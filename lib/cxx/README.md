# CSI C++ Validation / Binding

This package is used to validate (or develop) the CSI Specification with C++ language bindings.

## Build Reference

```bash
$ make
```

This command will download `protoc` and build the CSI protobuf for C++.

### Environment Variables

The following table lists the environment variables that can be used to influence the behavior of the Makefile:

| Name | Default Value | Description |
|------|---------------|-------------|
| `PROTOC_VER` | `24.0` | The version of the protoc binary. |
