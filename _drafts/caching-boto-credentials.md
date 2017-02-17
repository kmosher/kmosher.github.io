---
title:  "Caching Assume-Role Sessions With Boto (Just Like the AWS CLI)"
layout: post
---

I'm very happy with credential handling in modern versions of [boto][boto].
Because I'm responsible for a dozen different AWS accounts,
I particularly like using the profiles feature in the [shared credentials file][shared-creds].
Furthemore, I like using `sts assume-role` (with MFA required) to secure access to sensitive AWS APIs.
Boto has good support for [this pattern][assume-role].
If you include a `source_profile` and `mfa_serial` in your profile entry,
it'll automatically make the STS call and prompt you for your MFA code.
This all acts exactly like the `aws` CLI tool because that tool is developed in close coöperation with boto.
In fact, the assume-role provider was originally developed in the awscli project [before being moved to botocore][assume-role-port].

There's just one _tiny_ difference.
When you use the `aws` tool with the assume-role provider,
it'll cache your STS credentials in your home directory and reuse them on future invocations until they expire.
Boto does not do this by default.
If you write a CLI script using boto, it will re-prompt you on every invocation for your MFA code.
This gets old _fast_.

Luckily, it's prett easy to mimic over the `aws` CLI's behavior.
Although, it's not so easy that I didn't feel the need to write this post explaining _how_.

When the assume-role provider was ported, it came with a ready-to-use `cache` member that supports any psuedo-dict object with `__contains__`, `__getitem__`, and `__setitem__`.
Setting the cache member is a little annoying because the provider lives on the underlying `botocore.Session` and not the `boto3.Session` object.
Nonetheless, it's not a terrible amount of boilerplate.

Here's some example code:

{% highlight python %}
def init_session(profile=None)
    core_session = botocore.session.Session(profile=profile)
    creds_component = core_session.get_component('credential_provider')
    provider = creds_component.get_provider('assume-role')
    provider.cache = JSONFileCache()
    session = boto3.Session(botocore_session=core_session)
    return session
{% endhighlight %}

Of course, since I want this to work just like the `aws` tool and even share the same cache files,
let's just go ahead and copy over the [JSONFileCache from `awcli`][json-file-cache].

{% highlight python %}
import os
import json

class JSONFileCache(object):
    """JSON file cache.
    This provides a dict like interface that stores JSON serializable
    objects.
    The objects are serialized to JSON and stored in a file.  These
    values can be retrieved at a later time.
    """

    # Change this if you don't want to share a cache with the AWS tool
    CACHE_DIR = os.path.expanduser(os.path.join('~', '.aws', 'cli', 'cache'))

    def __init__(self, working_dir=CACHE_DIR):
        self._working_dir = working_dir

    def __contains__(self, cache_key):
        actual_key = self._convert_cache_key(cache_key)
        return os.path.isfile(actual_key)

    def __getitem__(self, cache_key):
        """Retrieve value from a cache key."""
        actual_key = self._convert_cache_key(cache_key)
        try:
            with open(actual_key) as f:
                return json.load(f)
        except (OSError, ValueError, IOError):
            raise KeyError(cache_key)

    def __setitem__(self, cache_key, value):
        full_key = self._convert_cache_key(cache_key)
        try:
            file_content = json.dumps(value)
        except (TypeError, ValueError):
            raise ValueError("Value cannot be cached, must be "
                             "JSON serializable: %s" % value)
        if not os.path.isdir(self._working_dir):
            os.makedirs(self._working_dir)
        with os.fdopen(os.open(full_key,
                               os.O_WRONLY | os.O_CREAT, 0o600), 'w') as f:
            f.truncate()
            f.write(file_content)

    def _convert_cache_key(self, cache_key):
        full_path = os.path.join(self._working_dir, cache_key + '.json')
        return full_path
{% endhighlight %}

If you try to set this as a cache on your session and then use the assume-role provider though, your script will explode with a `ValueError`.
The `Expiration` field on the credentials being cached will be a `datetime.datetime`, which can't be JSON serialized.
Oops.

So, why doesn't `awcli` have this problem?
Timestamps returned by the AWS API are simply returned as ISO-8061 strings.
Boto, though, helpfully parses timestamps to more useful datetime objects.
This behavior is, however, controllable, and [`awscli` overrides it to keep timestamps as strings][aws-timestamp-parsing].

We could also modify timestamp processing on our session to return them as strings,
but that's unnecessarily limiting.
Having timestamps as native datetime objects is useful!
Furthermore, maybe I want to go the opposite direction and return timestamps as awesome [arrow][arrow] objects.

The better fix is to change the serialization to support a wider range of timestamps.
Luckily, it's easy to extend the json serializer with a fallback serialization function.

{% highlight python %}
def serialize_timestamps(obj):
    """Serializes timestamps objects to ISO 8061 strings"""
    try:
        return obj.isoformat()
    except:
        raise TypeError (f"Type {type(obj)} not serializable")

json.dumps(value, default=serialize_timestamps)
{% endhighlight %}

Once you update the `__setitem__` function on `JSONFileCache` to use the fallback serialization, it should all start working.
With this in place, any scripts you write with this code will all share the same session cache with both eachother and with `aws`.
That's pretty sweet, and excatly the behavior I want my tools to have.

Happy boto scripting.


[boto]: https://boto3.readthedocs.io/en/latest/index.html
[shared-creds]: https://boto3.readthedocs.io/en/latest/guide/configuration.html#shared-credentials-file
[assume-role]: https://boto3.readthedocs.io/en/latest/guide/configuration.html#assume-role-provider
[assume-role-port]: https://github.com/boto/botocore/commit/2dae76f52ae63db3304b5933730ea5efaaaf2bfc#diff-dcd72ae45553af71f1ceafb2d4ee5dcf
[json-file-cache]: https://github.com/aws/aws-cli/blob/ad914b745ebcae76186512d91ed9e817be29bd46/awscli/customizations/assumerole.py
[aws-timestamp-parsing]: https://github.com/aws/aws-cli/blob/ad914b745ebcae76186512d91ed9e817be29bd46/awscli/customizations/scalarparse.py
[arrow]: http://crsmithdev.com/arrow/