# Creating CDP access keys

To use the CDP CLI & CDP REST APIs, it is necessary to generate CDP access keys.  Here is how you do it.

1. From the CDP Console, go to User Management
2. Find your user, and click into it
3. Under Actions, click Generate Access Key
4. Click the Generate Access Key button

This will bring up a pop-up with your Access Key ID and Private Key, and give you the option to download your credentials file.  You should totally do that now. 

The private key will not be recoverable if you don't download the credentials file at this time.

Your access key id will be of the form
>>
`5e4865c9-1234-47fb-a1a1-4ecc5de6d9ec`

Your private key will be of this form, noting that it may end with an "=" which is part of the private key.

>>
`KV8llPjt64PpZ1234Z/r/jcjmRU8RuDSQBdgvKV8x+km8=`

# Configuring the CDP CLI

From a terminal window, find your credentials file you just downloaded (it's probably called `credentials`).   Cat it so you can see the values you'll use in a moment.

>>
```
cat ~/Downloads/credentials
[default]
cdp_access_key_id=5e4865c9-1234-47fb-a1a1-4ecc5de6d9ec
cdp_private_key=KV8llPjt64PpZ1234Z/r/jcjmRU8RuDSQBdgvKV8x+km8=%
```

Note that the credentials file adds a `%` to the private key.   _*I think this needs to be ignored, but I need to re-verify.*_


Now run `cdp configure` and copy/paste the values from the credentials file.
