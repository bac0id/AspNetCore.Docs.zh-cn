---
title: 创建 gRPC 服务和方法
author: jamesnk
description: 了解如何创建 gRPC 服务和方法。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/25/2020
no-loc:
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: grpc/services
ms.openlocfilehash: fde589b2de9d908db26a2557c5f57c715625aadc
ms.sourcegitcommit: 47c9a59ff8a359baa6bca2637d3af87ddca1245b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2020
ms.locfileid: "88945452"
---
# <a name="create-grpc-services-and-methods"></a><span data-ttu-id="211db-103">创建 gRPC 服务和方法</span><span class="sxs-lookup"><span data-stu-id="211db-103">Create gRPC services and methods</span></span>

<span data-ttu-id="211db-104">本文档介绍如何以 C# 创建 gRPC 服务和方法。</span><span class="sxs-lookup"><span data-stu-id="211db-104">This document explains how to create gRPC services and methods in C#.</span></span> <span data-ttu-id="211db-105">主题包括：</span><span class="sxs-lookup"><span data-stu-id="211db-105">Topics include:</span></span>

* <span data-ttu-id="211db-106">如何在 .proto 文件中定义服务和方法。</span><span class="sxs-lookup"><span data-stu-id="211db-106">How to define services and methods in *.proto* files.</span></span>
* <span data-ttu-id="211db-107">使用 gRPC C# 工具生成的代码。</span><span class="sxs-lookup"><span data-stu-id="211db-107">Generated code using gRPC C# tooling.</span></span>
* <span data-ttu-id="211db-108">实现 gRPC 服务和方法。</span><span class="sxs-lookup"><span data-stu-id="211db-108">Implementing gRPC services and methods.</span></span>

## <a name="create-new-grpc-services"></a><span data-ttu-id="211db-109">创建新的 gRPC 服务</span><span class="sxs-lookup"><span data-stu-id="211db-109">Create new gRPC services</span></span>

<span data-ttu-id="211db-110">[使用 C# 的 gRPC 服务](xref:grpc/basics)介绍了 gRPC 的 API 开发协定优先方法。</span><span class="sxs-lookup"><span data-stu-id="211db-110">[gRPC services with C#](xref:grpc/basics) introduced gRPC's contract-first approach to API development.</span></span> <span data-ttu-id="211db-111">在 .proto 文件中定义服务和消息。</span><span class="sxs-lookup"><span data-stu-id="211db-111">Services and messages are defined in *.proto* files.</span></span> <span data-ttu-id="211db-112">然后，C# 工具从 .proto 文件生成代码。</span><span class="sxs-lookup"><span data-stu-id="211db-112">C# tooling then generates code from *.proto* files.</span></span> <span data-ttu-id="211db-113">对于服务器端资产，将为每个服务生成一个抽象基类型，同时为所有消息生成类。</span><span class="sxs-lookup"><span data-stu-id="211db-113">For server-side assets, an abstract base type is generated for each service, along with classes for any messages.</span></span>

<span data-ttu-id="211db-114">以下 .proto 文件：</span><span class="sxs-lookup"><span data-stu-id="211db-114">The following *.proto* file:</span></span>

* <span data-ttu-id="211db-115">定义 `Greeter` 服务。</span><span class="sxs-lookup"><span data-stu-id="211db-115">Defines a `Greeter` service.</span></span>
* <span data-ttu-id="211db-116">`Greeter` 服务定义 `SayHello` 调用。</span><span class="sxs-lookup"><span data-stu-id="211db-116">The `Greeter` service defines a `SayHello` call.</span></span>
* <span data-ttu-id="211db-117">`SayHello` 发送 `HelloRequest` 消息并接收 `HelloReply` 消息</span><span class="sxs-lookup"><span data-stu-id="211db-117">`SayHello` sends a `HelloRequest` message and receives a `HelloReply` message</span></span>

```protobuf
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

<span data-ttu-id="211db-118">C# 工具生成 C# `GreeterBase` 基类型：</span><span class="sxs-lookup"><span data-stu-id="211db-118">C# tooling generates the C# `GreeterBase` base type:</span></span>

```csharp
public abstract partial class GreeterBase
{
    public virtual Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        throw new RpcException(new Status(StatusCode.Unimplemented, ""));
    }
}

public class HelloRequest
{
    public string Name { get; set; }
}

public class HelloReply
{
    public string Message { get; set; }
}
```

<span data-ttu-id="211db-119">默认情况下，生成的 `GreeterBase` 不执行任何操作。</span><span class="sxs-lookup"><span data-stu-id="211db-119">By default the generated `GreeterBase` doesn't do anything.</span></span> <span data-ttu-id="211db-120">它的虚拟 `SayHello` 方法会将 `UNIMPLEMENTED` 错误返回到调用它的任何客户端。</span><span class="sxs-lookup"><span data-stu-id="211db-120">Its virtual `SayHello` method will return an `UNIMPLEMENTED` error to any clients that call it.</span></span> <span data-ttu-id="211db-121">为了使服务有用，应用必须创建 `GreeterBase` 的具体实现：</span><span class="sxs-lookup"><span data-stu-id="211db-121">For the service to be useful an app must create a concrete implementation of `GreeterBase`:</span></span>

```csharp
public class GreeterService : GreeterBase
{
    public override Task<HelloReply> UnaryCall(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloRequest { Message = $"Hello {request.Name}" });
    }
}
```

<span data-ttu-id="211db-122">服务实现已注册到应用。</span><span class="sxs-lookup"><span data-stu-id="211db-122">The service implementation is registered with the app.</span></span> <span data-ttu-id="211db-123">如果服务由 ASP.NET Core gRPC 托管，则应使用 `MapGrpcService` 方法将其添加到路由管道。</span><span class="sxs-lookup"><span data-stu-id="211db-123">If the service is hosted by ASP.NET Core gRPC, it should be added to the routing pipeline with the `MapGrpcService` method.</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<GreeterService>();
});
```

<span data-ttu-id="211db-124">有关更多信息，请参见 <xref:grpc/aspnetcore> 。</span><span class="sxs-lookup"><span data-stu-id="211db-124">See <xref:grpc/aspnetcore> for more information.</span></span>

## <a name="implement-grpc-methods"></a><span data-ttu-id="211db-125">实现 gRPC 方法</span><span class="sxs-lookup"><span data-stu-id="211db-125">Implement gRPC methods</span></span>

<span data-ttu-id="211db-126">gRPC 服务可以有不同类型的方法。</span><span class="sxs-lookup"><span data-stu-id="211db-126">A gRPC service can have different types of methods.</span></span> <span data-ttu-id="211db-127">服务发送和接收消息的方式取决于所定义的方法的类型。</span><span class="sxs-lookup"><span data-stu-id="211db-127">How messages are sent and received by a service depends on the type of method defined.</span></span> <span data-ttu-id="211db-128">gRPC 方法类型如下：</span><span class="sxs-lookup"><span data-stu-id="211db-128">The gRPC method types are:</span></span>

* <span data-ttu-id="211db-129">一元</span><span class="sxs-lookup"><span data-stu-id="211db-129">Unary</span></span>
* <span data-ttu-id="211db-130">服务器流式处理</span><span class="sxs-lookup"><span data-stu-id="211db-130">Server streaming</span></span>
* <span data-ttu-id="211db-131">客户端流式处理</span><span class="sxs-lookup"><span data-stu-id="211db-131">Client streaming</span></span>
* <span data-ttu-id="211db-132">双向流式处理</span><span class="sxs-lookup"><span data-stu-id="211db-132">Bi-directional streaming</span></span>

<span data-ttu-id="211db-133">流式处理调用是使用 `stream` 关键字在 .proto 文件中指定的。</span><span class="sxs-lookup"><span data-stu-id="211db-133">Streaming calls are specified with the `stream` keyword in the *.proto* file.</span></span> <span data-ttu-id="211db-134">`stream` 可以放置在调用的请求消息和/或响应消息中。</span><span class="sxs-lookup"><span data-stu-id="211db-134">`stream` can be placed on a call's request message, response message, or both.</span></span>

```protobuf
syntax = "proto3";

service ExampleService {
  // Unary
  rpc UnaryCall (ExampleRequest) returns (ExampleResponse);

  // Server streaming
  rpc StreamingFromServer (ExampleRequest) returns (stream ExampleResponse);

  // Client streaming
  rpc StreamingFromClient (stream ExampleRequest) returns (ExampleResponse);

  // Bi-directional streaming
  rpc StreamingBothWays (stream ExampleRequest) returns (stream ExampleResponse);
}
```

<span data-ttu-id="211db-135">每个调用类型都有不同的方法签名。</span><span class="sxs-lookup"><span data-stu-id="211db-135">Each call type has a different method signature.</span></span> <span data-ttu-id="211db-136">在具体实现中替代从抽象基本服务类型生成的方法，可确保使用正确的参数和返回类型。</span><span class="sxs-lookup"><span data-stu-id="211db-136">Overriding generated methods from the abstract base service type in a concrete implementation ensures the correct arguments and return type are used.</span></span>

### <a name="unary-method"></a><span data-ttu-id="211db-137">一元方法</span><span class="sxs-lookup"><span data-stu-id="211db-137">Unary method</span></span>

<span data-ttu-id="211db-138">一元方法以参数的形式获取请求消息，并返回响应。</span><span class="sxs-lookup"><span data-stu-id="211db-138">A unary method gets the request message as a parameter, and returns the response.</span></span> <span data-ttu-id="211db-139">返回响应时，一元调用完成。</span><span class="sxs-lookup"><span data-stu-id="211db-139">A unary call is complete when the response is returned.</span></span>

```csharp
public override Task<ExampleResponse> UnaryCall(ExampleRequest request,
    ServerCallContext context)
{
    var response = new ExampleResponse();
    return Task.FromResult(response);
}
```

<span data-ttu-id="211db-140">一元调用与 [Web API 控制器上的操作](xref:web-api/index)最为相似。</span><span class="sxs-lookup"><span data-stu-id="211db-140">Unary calls are the most similar to [actions on web API controllers](xref:web-api/index).</span></span> <span data-ttu-id="211db-141">gRPC 方法与操作的一个重要区别是，gRPC 方法无法将请求的某些部分绑定到不同的方法参数。</span><span class="sxs-lookup"><span data-stu-id="211db-141">One important difference gRPC methods have from actions is gRPC methods are not able to bind parts of a request to different method arguments.</span></span> <span data-ttu-id="211db-142">对于传入请求数据，gRPC 方法始终有一个消息参数。</span><span class="sxs-lookup"><span data-stu-id="211db-142">gRPC methods always have one message argument for the incoming request data.</span></span> <span data-ttu-id="211db-143">通过在请求消息中设置多个值字段，仍可以将多个值发送到 gRPC 服务：</span><span class="sxs-lookup"><span data-stu-id="211db-143">Multiple values can still be sent to a gRPC service by making them fields on the request message:</span></span>

```protobuf
message ExampleRequest {
    int pageIndex = 1;
    int pageSize = 2;
    bool isDescending = 3;
}
```

### <a name="server-streaming-method"></a><span data-ttu-id="211db-144">服务器流式处理方法</span><span class="sxs-lookup"><span data-stu-id="211db-144">Server streaming method</span></span>

<span data-ttu-id="211db-145">服务器流式处理方法以参数的形式获取请求消息。</span><span class="sxs-lookup"><span data-stu-id="211db-145">A server streaming method gets the request message as a parameter.</span></span> <span data-ttu-id="211db-146">由于可以将多个消息流式传输回调用方，因此可使用 `responseStream.WriteAsync` 发送响应消息。</span><span class="sxs-lookup"><span data-stu-id="211db-146">Because multiple messages can be streamed back to the caller, `responseStream.WriteAsync` is used to send response messages.</span></span> <span data-ttu-id="211db-147">当方法返回时，服务器流式处理调用完成。</span><span class="sxs-lookup"><span data-stu-id="211db-147">A server streaming call is complete when the method returns.</span></span>

```csharp
public override async Task StreamingFromServer(ExampleRequest request,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    for (var i = 0; i < 5; i++)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1));
    }
}
```

<span data-ttu-id="211db-148">服务器流式处理方法启动后，客户端无法发送其他消息或数据。</span><span class="sxs-lookup"><span data-stu-id="211db-148">The client has no way to send additional messages or data once the server streaming method has started.</span></span> <span data-ttu-id="211db-149">某些流式处理方法设计为永久运行。</span><span class="sxs-lookup"><span data-stu-id="211db-149">Some streaming methods are designed to run forever.</span></span> <span data-ttu-id="211db-150">对于连续流式处理方法，客户端可以在不再需要调用时将其取消。</span><span class="sxs-lookup"><span data-stu-id="211db-150">For continuous streaming methods, a client can cancel the call when it's no longer needed.</span></span> <span data-ttu-id="211db-151">当发生取消时，客户端会将信号发送到服务器，并引发 [ServerCallContext.CancellationToken](xref:System.Threading.CancellationToken)。</span><span class="sxs-lookup"><span data-stu-id="211db-151">When cancellation happens the client sends a signal to the server and the [ServerCallContext.CancellationToken](xref:System.Threading.CancellationToken) is raised.</span></span> <span data-ttu-id="211db-152">应在服务器上通过异步方法使用 `CancellationToken` 标记，以实现以下目的：</span><span class="sxs-lookup"><span data-stu-id="211db-152">The `CancellationToken` token should be used on the server with async methods so that:</span></span>

* <span data-ttu-id="211db-153">所有异步工作都与流式处理调用一起取消。</span><span class="sxs-lookup"><span data-stu-id="211db-153">Any asynchronous work is canceled together with the streaming call.</span></span>
* <span data-ttu-id="211db-154">该方法快速退出。</span><span class="sxs-lookup"><span data-stu-id="211db-154">The method exits quickly.</span></span>

```csharp
public override async Task StreamingFromServer(ExampleRequest request,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    while (!context.CancellationToken.IsCancellationRequested)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1), context.CancellationToken);
    }
}
```

### <a name="client-streaming-method"></a><span data-ttu-id="211db-155">客户端流式处理方法</span><span class="sxs-lookup"><span data-stu-id="211db-155">Client streaming method</span></span>

<span data-ttu-id="211db-156">客户端流式处理方法在该方法没有接收消息的情况下启动。</span><span class="sxs-lookup"><span data-stu-id="211db-156">A client streaming method starts *without* the method receiving a message.</span></span> <span data-ttu-id="211db-157">`requestStream` 参数用于从客户端读取消息。</span><span class="sxs-lookup"><span data-stu-id="211db-157">The `requestStream` parameter is used to read messages from the client.</span></span> <span data-ttu-id="211db-158">返回响应消息时，客户端流式处理调用完成：</span><span class="sxs-lookup"><span data-stu-id="211db-158">A client streaming call is complete when a response message is returned:</span></span>

```csharp
public override async Task<ExampleResponse> StreamingFromClient(
    IAsyncStreamReader<ExampleRequest> requestStream, ServerCallContext context)
{
    while (await requestStream.MoveNext())
    {
        var message = requestStream.Current;
        // ...
    }
    return new ExampleResponse();
}
```

<span data-ttu-id="211db-159">如果使用 C# 8 或更高版本，则可使用 `await foreach` 语法来读取消息。</span><span class="sxs-lookup"><span data-stu-id="211db-159">When using C# 8 or later, the `await foreach` syntax can be used to read messages.</span></span> <span data-ttu-id="211db-160">`IAsyncStreamReader<T>.ReadAllAsync()` 扩展方法读取请求数据流中的所有消息：</span><span class="sxs-lookup"><span data-stu-id="211db-160">The `IAsyncStreamReader<T>.ReadAllAsync()` extension method reads all messages from the request stream:</span></span>

```csharp
public override async Task<ExampleResponse> StreamingFromClient(
    IAsyncStreamReader<ExampleRequest> requestStream, ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        // ...
    }
    return new ExampleResponse();
}
```

### <a name="bi-directional-streaming-method"></a><span data-ttu-id="211db-161">双向流式处理方法</span><span class="sxs-lookup"><span data-stu-id="211db-161">Bi-directional streaming method</span></span>

<span data-ttu-id="211db-162">双向流式处理方法在该方法没有接收到消息的情况下启动。</span><span class="sxs-lookup"><span data-stu-id="211db-162">A bi-directional streaming method starts *without* the method receiving a message.</span></span> <span data-ttu-id="211db-163">`requestStream` 参数用于从客户端读取消息。</span><span class="sxs-lookup"><span data-stu-id="211db-163">The `requestStream` parameter is used to read messages from the client.</span></span> <span data-ttu-id="211db-164">该方法可选择使用 `responseStream.WriteAsync` 发送消息。</span><span class="sxs-lookup"><span data-stu-id="211db-164">The method can choose to send messages with `responseStream.WriteAsync`.</span></span> <span data-ttu-id="211db-165">当方法返回时，双向流式处理调用完成：</span><span class="sxs-lookup"><span data-stu-id="211db-165">A bi-directional streaming call is complete when the the method returns:</span></span>

```csharp
public override async Task StreamingBothWays(IAsyncStreamReader<ExampleRequest> requestStream,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        await responseStream.WriteAsync(new ExampleResponse());
    }
}
```

<span data-ttu-id="211db-166">前面的代码：</span><span class="sxs-lookup"><span data-stu-id="211db-166">The preceding code:</span></span>

* <span data-ttu-id="211db-167">发送每个请求的响应。</span><span class="sxs-lookup"><span data-stu-id="211db-167">Sends a response for each request.</span></span>
* <span data-ttu-id="211db-168">是双向流式处理的基本用法。</span><span class="sxs-lookup"><span data-stu-id="211db-168">Is a basic usage of bi-directional streaming.</span></span>

<span data-ttu-id="211db-169">可以支持更复杂的方案，例如同时读取请求和发送响应：</span><span class="sxs-lookup"><span data-stu-id="211db-169">It is possible to support more complex scenarios, such as reading requests and sending responses simultaneously:</span></span>

```csharp
public override async Task StreamingBothWays(IAsyncStreamReader<ExampleRequest> requestStream,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    // Read requests in a background task.
    var readTask = Task.Run(async () =>
    {
        await foreach (var message in requestStream.ReadAllAsync())
        {
            // Process request.
        }
    });
    
    // Send responses until the client signals that it is complete.
    while (!readTask.IsCompleted)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1), context.CancellationToken);
    }
}
```

<span data-ttu-id="211db-170">在双向流式处理方法中，客户端和服务可在任何时间互相发送消息。</span><span class="sxs-lookup"><span data-stu-id="211db-170">In a bi-directional streaming method, the client and service can send messages to each other at any time.</span></span> <span data-ttu-id="211db-171">双向方法的最佳实现根据需求而有所不同。</span><span class="sxs-lookup"><span data-stu-id="211db-171">The best implementation of a bi-directional method varies depending upon requirements.</span></span>

## <a name="access-grpc-request-headers"></a><span data-ttu-id="211db-172">访问 gRPC 请求标头</span><span class="sxs-lookup"><span data-stu-id="211db-172">Access gRPC request headers</span></span>

<span data-ttu-id="211db-173">请求消息并不是客户端将数据发送到 gRPC 服务的唯一方法。</span><span class="sxs-lookup"><span data-stu-id="211db-173">A request message is not the only way for a client to send data to a gRPC service.</span></span> <span data-ttu-id="211db-174">标头值在使用 `ServerCallContext.RequestHeaders` 的服务中可用。</span><span class="sxs-lookup"><span data-stu-id="211db-174">Header values are available in a service using `ServerCallContext.RequestHeaders`.</span></span>

```csharp
public override Task<ExampleResponse> UnaryCall(ExampleRequest request, ServerCallContext context)
{
    var userAgent = context.RequestHeaders.GetValue("user-agent");
    // ...

    return Task.FromResult(new ExampleResponse());
}
```

## <a name="additional-resources"></a><span data-ttu-id="211db-175">其他资源</span><span class="sxs-lookup"><span data-stu-id="211db-175">Additional resources</span></span>

* <xref:grpc/basics>
* <xref:grpc/client>