# Build in a .NET Core SDK container

You can use Docker to run your build in an isolated environment using the [.NET Core SDK Docker image](https://hub.docker.com/_/microsoft-dotnet-core-sdk/). This is useful to either avoid the need to install .NET Core on the build machine or ensure that your environment is correctly configured (dev, staging, or production).

The instructions assume that you have cloned the repository locally, and that you are in the `samples/dotnetapp` directory (due to the volume mounting syntax), as demonstrated by the examples.

## Requirements

This scenario relies on [volume mounting](https://docs.docker.com/engine/admin/volumes/volumes/) (that's the `-v` argument) to make source available within the container (to build it). You may need to [Enable shared drives (Windows)](https://docs.docker.com/docker-for-windows/#shared-drives) or [file sharing (macOS)](https://docs.docker.com/docker-for-mac/#file-sharing) first.

`dotnet publish` (and `build`) produces native executables for applications. If you use a Linux container, you will build a Linux executable that will not run on Windows or macOS. You can use a runtime argument (`-r`) to specify the type of assets that you want to publish (if they don't match the SDK container). The following examples assume you want assets that match your host operating system, and use runtime arguments to ensure that.

## Pull SDK image

It is recommended to pull the SDK image before running the appropriate command. This ensures that you get the latest patch version of the SDK. Use the following command:

```console
docker pull mcr.microsoft.com/dotnet/core/sdk:3.1
```

This step ensures you have the latest patch of the SDK image on your machine.

## Linux

```console
docker run --rm -v $(pwd):/app -w /app mcr.microsoft.com/dotnet/core/sdk:3.1 dotnet publish -c release -o out
```

You can see the built binaries with the following command:

```console
rich@thundera dotnetapp % ls out
dotnetapp			    dotnetapp.pdb
dotnetapp.deps.json		dotnetapp.runtimeconfig.json
dotnetapp.dll
```

## macOS

```console
docker run --rm -v $(pwd):/app -w /app mcr.microsoft.com/dotnet/core/sdk:3.1 dotnet publish -c release -o out -r osx-x64 --self-contained false
```

You can see the built binaries with the following command:

```console
rich@thundera dotnetapp % ls out
dotnetapp			    dotnetapp.pdb
dotnetapp.deps.json		dotnetapp.runtimeconfig.json
dotnetapp.dll
```

## Windows using Linux containers

```console
docker run --rm -v %cd%:/app -w /app mcr.microsoft.com/dotnet/core/sdk:3.1 dotnet publish -c release -o out -r win-x64 --self-contained false
```

## Windows using Windows containers

```console
docker run --rm -v %cd%:c:\app -w c:\app mcr.microsoft.com/dotnet/core/sdk:3.1 dotnet publish -c Release -o out
```

## Building to a separate location

You may want the build output to be written to a separate location than the source directory. That's easy to do with a second volume mount. 

The following example demonstrates doing that on macOS:

```console
docker run --rm -v ~/dotnetapp:/out -v $(pwd):/app -w /app mcr.microsoft.com/dotnet/core/sdk:3.1 dotnet publish -c release -o /out -r osx-x64 --self-contained false
```

You can see the built binaries with the following command:

```console
> ls ~/dotnetapp
dotnetapp			    dotnetapp.pdb
dotnetapp.deps.json		dotnetapp.runtimeconfig.json
dotnetapp.dll
```

The following example demonstrates doing that on Windows (using Linux containers):

```console
docker run --rm -v C:\dotnetapp:/out -v %cd%:/app -w /app mcr.microsoft.com/dotnet/core/sdk:3.1 dotnet publish -c release -o /out -r win-x64 --self-contained false
```

## Resources

* [.NET Core Docker Samples](../README.md)
* [.NET Framework Docker Samples](https://github.com/microsoft/dotnet-framework-docker/blob/master/samples/README.md)