<font size="6" style="font-weight: bold;"> ⚠️ Notice: Limited Maintenance </font>

This project is no longer actively maintained. While existing releases remain available, there are no planned updates, bug fixes, new features, or security patches. Users should be aware that vulnerabilities may not be addressed.

# TorchServe token authorization API

TorchServe now enforces token authorization by default

TorchServe enforces token authorization by default which requires the correct token to be provided when calling an API. This is a security feature which addresses the concern of unauthorized API calls. This is applicable in the scenario where an unauthorized user may try to access a running TorchServe instance. The default behavior is to enable this feature which creates a key file with the appropriate tokens to be used for API calls. Users can disable this feature to prevent token authorization from being required for API calls ([how to disable](#how-to-set-and-disable-token-authorization)), however users are warned that this will open up TorchServe to potential unauthorized API calls.

## How to set and disable Token Authorization
* Global environment variable: use `TS_DISABLE_TOKEN_AUTHORIZATION` and set to `true` to disable and `false` to enable token authorization. Note that `enable_envvars_config=true` must be set in config.properties for global environment variables to be used
* Command line: Command line can only be used to disable token authorization by adding the `--disable-token-auth` flag.
* Config properties file: use `disable_token_authorization` and set to `true` to disable and `false` to enable token authorization.

Priority between env variables, cmd, and config file follows the following [TorchServer standard](https://github.com/pytorch/serve/blob/master/docs/configuration.md)

* Example 1:
  * Config file: `disable_token_authorization=false`

    cmd line: `torchserve --start --ncs --model-store model_store --disable-token-auth`

    Result: Token authorization disabled through command line but enabled through config file, resulting in token authorization being disabled. Command line takes precedence
* Example 2:
  * Config file: `disable_token_authorization=true`

    cmd line: `torchserve --start --ncs --model-store model_store`

    Result: Token authorization disable disabled through config file but not configured through command line, resulting in token authorization being disabled.

## Configuration
1. Torchserve will enable token authorization by default. Expected log statement `main org.pytorch.serve.http.TokenAuthorizationHandler - Token Authorization Enabled`
2. In the current working directory a file `key_file.json` will be generated.
    1. Example key file:

```python
  {
  "management": {
    "key": "B-E5KSRM",
    "expiration time": "2024-02-16T21:12:24.801167Z"
  },
  "inference": {
    "key": "gNRuA7dS",
    "expiration time": "2024-02-16T21:12:24.801148Z"
  },
  "API": {
    "key": "yv9uQajP"
  }
}
```

3. There are 3 keys and each have a different use.
    1. Management key: Used for management APIs. Example:
    `curl http://localhost:8081/models/densenet161 -H "Authorization: Bearer I_J_ItMb"`
    2. Inference key: Used for inference APIs. Example:
    `curl http://127.0.0.1:8080/predictions/densenet161 -T examples/image_classifier/kitten.jpg -H "Authorization: Bearer FINhR1fj"`
    3. API key: Used for the token authorization API. Check section 4 for API use.
4. API in order to generate a new key to replace either the management or inference key.
    1. Management Example:
    `curl localhost:8081/token?type=management -H "Authorization: Bearer m4M-5IBY"` will replace the current management key in the key_file with a new one and will update the expiration time.
    2. Inference example:
    `curl localhost:8081/token?type=inference -H "Authorization: Bearer m4M-5IBY"`

    Users will have to use either one of the APIs above.

5. When users shut down the server the key_file will be deleted.

## Notes
1. DO NOT MODIFY THE KEY FILE. Modifying the key file might impact reading and writing to the file thus preventing new keys from properly being displayed in the file.
2. Time to expiration is set to default at 60 minutes but can be changed in the config.properties by adding `token_expiration_min`. Ex:`token_expiration_min=30`
3. Three tokens allow the owner with the most flexibility in use and enables them to adapt the tokens to their use. Owners of the server can provide users with the inference token if users should only be able to run inferences against models that have already been loaded. The owner can also provide owners with the management key if owners want users to add and remove models.
