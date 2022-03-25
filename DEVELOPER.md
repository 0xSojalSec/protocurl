# Developer

Almost all development, except for the execution of `test/suite/test.sh` is based on docker. For the execution of the
test script, one can use native bash or attempt to use WSL and variations on Windows.

## Test server

When new dependencies are needed for the test server, the following command enables one to start a shell in the test
server.

```
docker run -v "$PWD/test/servers:/servers" -it nodeserver:v1 /bin/bash
```

Now it's possible to add new dependencies via `npm install <new-package>`