# smooth-secrets-buildkite-plugin
A plugin to setup ssh keys and env secrets before checking out your code.

## Configuration
```yml
secrets:
  - strategy:
    region:
    key:
    type:
```

### `secrets` (array)
- #### `strategy` (required, string)
    Supported value: `aws-secrets-manager`
- #### `key` (required, string)
    Secret id to refer to the secret in aws secrets manager
- #### `type` (required, string)
    Supported value: `ssh`  
    Will add the secret value as a private ssh key to the ssh-agent
- #### `region` (required, string)
    Region value for aws

## Usage

```yml
steps:
  - command: ssh-add -l
    plugins:
      - hasura/smooth-secrets#v1.0.0:
          secrets:
            - strategy: aws-secrets-manager
              region: us-east-2
              key: secret/id
              type: ssh
```

## **Notes**
- This plugin does not take care of any kind of authentication with the secrets manager.
- The ssh private key is stored in directory `/etc/buildkite-agent/buildkite-secrets/${BUILDKITE_BUILD_ID}/${BUILDKITE_JOB_ID}`. 
The filename is the `key` field value with any `/` replaced with `-`.
- The keys are added to a newly created `ssh-agent`, which is killed at the end of the job in `pre-exit` hook. 
- The secrets directory is also removed in the `pre-exit` hook.
