# smooth-secrets-buildkite-plugin
A buildkite plugin to setup ssh keys and env secrets for your pipelines :butter: :lock:

## Usage

#### Exporting secrets to environment
```yml
steps:
  - command: echo "\$SECRET_NAME" > secret.txt
    plugins:
      - hasura/smooth-secrets#v1.2.0:
          secrets:
            - strategy: aws-secrets-manager
              region: us-east-2
              key: secret/env
              name: SECRET_NAME
              type: env
```

#### Creating a file with the secret contents
```yml
steps:
  - command: cd "$(dirname "${FILE_PATH_ENV}")" && cat "${FILE_NAME_ENV}"
    plugins:
      - hasura/smooth-secrets#v1.2.0:
          secrets:
            - strategy: aws-secrets-manager
              region: us-east-2
              key: secret/id
              type: file
              file_path_env: FILE_PATH_ENV
              file_name_env: FILE_NAME_ENV
```
The path at which the file is created will be exported to the environment with the name given in `file_path_env` field. Likewise, the file name will be exported with the name given in `file_name_env`. For example, here, `FILE_PATH_ENV` var will be set to the file path and `FILE_NAME_ENV` will be set to the filename.

#### Adding an SSH key to `ssh-agent`
```yml
steps:
  - command: ssh-add -l
    plugins:
      - hasura/smooth-secrets#v1.2.0:
          secrets:
            - strategy: aws-secrets-manager
              region: us-east-2
              key: secret/id
              type: ssh
```
- smooth-secret expects the private SSH key to be stored as base64 value in the secrets manager. Use `cat <KEY_FILE_PATH> | base64 -w 0` to get the base64 value.
- The private SSH key is stored in this directory: `/etc/buildkite-agent/buildkite-secrets/${BUILDKITE_BUILD_ID}/${BUILDKITE_JOB_ID}`. The filename is the `key` field value with any `/` replaced with `-`.
- The keys are added to a newly created `ssh-agent`, which is killed at the end of the job in `pre-exit` hook. 
- The secrets directory is also removed in the `pre-exit` hook.

#### Exporting base64 decoded secrets to environment
If the secret is stored as base64 encoded value in the secret storage, then smooth-secret can automatically decode and populate such secrets via the `encoding` field.
```yml
steps:
  - command: ssh-add -l
    plugins:
      - hasura/smooth-secrets#v1.2.0:
          secrets:
            - strategy: aws-secrets-manager
              region: us-east-2
              key: secret/env
              name: SECRET_NAME
              type: env
              encoding: base64
```

## Configuration

### `secrets` (array)
- #### `strategy` (required, string)
    Supported value: `aws-secrets-manager`
- #### `key` (required, string)
    Secret id to refer to the secret in the secret storage.
- #### `type` (required, string)
    Supported value: `ssh`, `env`, `file`
    - ssh will add the secret value as a private ssh key to the ssh-agent.
    - env will export the env for usage in the build.
    - file will create a file with the secret value as contents
- #### `name` (string)
    The name with which `env` type secrets will be exported.
    Only required when the secret type is `env`.
- #### `region` (required, string)
    Region value for aws
- #### `encoding` (optional, string)
    Supported value: `base64`
