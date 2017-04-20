---
title: Valet Key
description: Use a token or key that provides clients with restricted direct access to a specific resource or service.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [data-management, security]

---

# 발렛 키(Valet Key)

[!INCLUDE [header](../_includes/header.md)]

응용 프로그램으로부터 데이터 전송을 떠넘기기 위해서 클라이언트에 특정 리소스에 대한 제한적 직접 액세스 권한을 제공하는 토큰을 사용합니다. 클라우드 호스트된 저장소 시스템이나 큐를 사용하는 응용 프로그램에 특히 유용하며, 비용을 최소화하고 확장성과 성능을 최대화할 수 있습니다.

## 컨텍스트와 문제점

클라이언트 프로그램과 웹 브라우저는 종종 파일이나 데이터 스트림을 응용 프로그램 저장소에서 읽고, 저장소에 쓰는 작업이 필요합니다. 일반적으로 그런 응용 프로그램은 데이터 이동을 처리합니다 -  저장소에서 데이터를 가져오거나 클라이언트에 데이터를 스트리밍하거나 업로드된 스트림을 클라이언트에서 읽고 데이터 저장소에 저장하는 방법으로. 그러나, 이 접근 방식은 계산, 메모리, 대역과 같은 소중한 리소스를 소비합니다.

데이터 저장소는 응용 프로그램에 데이터의 이동 처리를 수행할 것을 요구하지 않고, 데이터를 직접 업로드하고 다운로드할 수 있는 능력이 있습니다. 그러나, 일반적으로 클라이언트가 저장소에 대한 보안 자격 증명을 액세스할 것을 요구합니다.  이는 데이터 전송 비용을 최소화하는 데 유용한 기법이며, 응용 프로그램을 확장하고 성능을 최대화하기 위한 요구 사항일 수 있습니다. 그렇다 해도, 응용 프로그램이 더 이상 데이터 보안을 관리할 수 없음을 의미합니다. 클라이언트가 직접 액세스를 위해 데이터 저장소에 접속한 이후에는, 응용 프로그램이 게이터키퍼 역할을 할 수 없습니다. 더 이상 프로세스를 제어하지 못하고, 데이터 저장소에서 그 다음 업로드나 다운로드 하는 것을 막을 수 없습니다.

신뢰 받지 않는 클라이언트에 도움이 되어야 하는 분산 시스템에 현실적인 접근 방식이 아닙니다. 대신, 응용 프로그램은 데이터 액세스를 자세히 안전하게 제어할 수 있어야 하며, 이렇게 연결을 설정한 다음 클라이언트가 직접 데이터 저장소와 통신하면서 필요한 읽기 또는 쓰기 작업을 수행하게 허용함으로써 서버에 대한 부하를 계속해서 줄여야 합니다.  

## 솔루션

저장소가 클라이언트의 인증과 권한 부여를 관리할 수 없는 데이터 저장소에 대한 액세스를 제어하는 문제를 해결해야 합니다. 일반적인 해결책은 데이터 저장소의 공용 연결에 대한 액세스를 제한하고 클라이언트에 데이터 저장소가 확인할 수 있는 키 또는 토큰을 제공하는 것입니다.

이것이 일반적으로 발렛 키라고 부르는 키 또는 토큰입니다. 발렛 키는 특정 리소스에 대한 시간 제한된 액세스를 제공하고, 저장소나 큐에 읽기와 쓰기 또는 웹 브라우저에 업로딩과 다운로딩과 같은 미리 정의된 작업 하나만 허용합니다. 응용 프로그램은 클라이언트 장치와 웹 브라우저에 대한 발렛 키를 빠르고 쉽게 만들고 발급할 수 있습니다. 이 키로 클라이언트는 응용 프로그램이 직접 데이터 전송을 처리할 것을 요구하지 않고 필요한 작업을 자신이 수행합니다. 이 경우, 응용 프로그램과 서버에서 일어나는 처리 오버헤드, 성능과 확장성에 미치는 영향을 제거합니다. 

그림과 같이, 클라이언트는 이 토큰을 사용하여 특정 기간에만, 액세스 사용 권한에 제한하여, 데이터 저장소에서 특정 리소스를 액세스합니다. 특정 기간이 지난 후, 키는 무효화되고 리소스에 대한 액세스를 허용하지 않습니다.

![Figure 1 - Overview of the pattern](./_images/valet-key-pattern.png)

데이터 범위와 같이 다른 종속성을 갖도록 키를 구성하는 것도 가능합니다. 예를 들면, 데이터 저장소 역량에 따라 키는 데이터 저장소에 완전한 표를 지정하거나 표에서 특정 행만 지정할 수 있습니다. 클라우드 저장소 시스템에서 키는 컨테이너, 즉 컨테이너 내 특정 항목을 지정할 수 있습니다.

키는 응용 프로그램에 의해 무효화될 수도 있습니다. 클라이언트가 서버에 데이터 전송 작업이 완료되었음을 알릴 경우, 이는 유용한 접근 방식입니다. 서버는 더 이상의 진행을 막기 위해 키를 무효화시킬 수 있습니다.

이 패턴을 사용하면 사용자 생성 및 인증, 사용 권한 부여, 사용자 다시 제거에 대한 요구 사항이 없기 때문에 리소스에 대한 액세스 관리가 단순화될 수 있습니다. 장소 제한, 사용 권한, 유효 기간을 제한하는 것도 쉬워졌습니다 - 모두 런타임에 키를 생성하여 가능함. 유효 기간, 특히 리소스의 위치를, 수신자만 그 용도에 맞게 사용할 수 있도록 가능한 한 엄격히 제한하는 것은 중요한 요소입니다. 

## 문제점 및 고려사항

이 패턴을 구현하는 방법을 결정할 때 다음 사항을 고려해야 합니다:

**키의 유효 상태와 기간 관리y**. If leaked or compromised, the key effectively unlocks the target item and makes it available for malicious use during the validity period. A key can usually be revoked or disabled, depending on how it was issued. Server-side policies can be changed or, the server key it was signed with can be invalidated. Specify a short validity period to minimize the risk of allowing unauthorized operations to take place against the data store. However, if the validity period is too short, the client might not be able to complete the operation before the key expires. Allow authorized users to renew the key before the validity period expires if multiple accesses to the protected resource are required.

**키가 제공하는 액세스 수준 제어**. Typically, the key should allow the user to only perform the actions necessary to complete the operation, such as read-only access if the client shouldn't be able to upload data to the data store. For file uploads, it's common to specify a key that provides write-only permission, as well as the location and the validity period. It's critical to accurately specify the resource or the set of resources to which the key applies.

**사용자 동작에 대한 제어 방법 고려**. Implementing this pattern means some loss of control over the resources users are granted access to. The level of control that can be exerted is limited by the capabilities of the policies and permissions available for the service or the target data store. For example, it's usually not possible to create a key that limits the size of the data to be written to storage, or the number of times the key can be used to access a file. This can result in huge unexpected costs for data transfer, even when used by the intended client, and might be caused by an error in the code that causes repeated upload or download. To limit the number of times a file can be uploaded, where possible, force the client to notify the application when one operation has completed. For example, some data stores raise events the application code can use to monitor operations and control user behavior. However, it's hard to enforce quotas for individual users in a multi-tenant scenario where the same key is used by all the users from one tenant.

**업로드된 모든 데이터의 확인 및 선택적 삭제**. A malicious user that gains access to the key could upload data designed to compromise the system. Alternatively, authorized users might upload data that's invalid and, when processed, could result in an error or system failure. To protect against this, ensure that all uploaded data is validated and checked for malicious content before use.

**모든 작업 감사**. Many key-based mechanisms can log operations such as uploads, downloads, and failures. These logs can usually be incorporated into an audit process, and also used for billing if the user is charged based on file size or data volume. Use the logs to detect authentication failures that might be caused by issues with the key provider, or accidental removal of a stored access policy.

**안전한 키 전달**. It can be embedded in a URL that the user activates in a web page, or it can be used in a server redirection operation so that the download occurs automatically. Always use HTTPS to deliver the key over a secure channel.

**전송 중인 중요한 데이터 보호**. Sensitive data delivered through the application will usually take place using SSL or TLS, and this should be enforced for clients accessing the data store directly.

Other issues to be aware of when implementing this pattern are:

- If the client doesn't, or can't, notify the server of completion of the operation, and the only limit is the expiration period of the key, the application won't be able to perform auditing operations such as counting the number of uploads or downloads, or preventing multiple uploads or downloads.

- The flexibility of key policies that can be generated might be limited. For example, some mechanisms only allow the use of a timed expiration period. Others aren't able to specify a sufficient granularity of read/write permissions.

- If the start time for the key or token validity period is specified, ensure that it's a little earlier than the current server time to allow for client clocks that might be slightly out of synchronization. The default, if not specified, is usually the current server time.

- The URL containing the key will be recorded in server log files. While the key will typically have expired before the log files are used for analysis, ensure that you limit access to them. If log data is transmitted to a monitoring system or stored in another location, consider implementing a delay to prevent leakage of keys until after their validity period has expired.

- If the client code runs in a web browser, the browser might need to support cross-origin resource sharing (CORS) to enable code that executes within the web browser to access data in a different domain from the one that served the page. Some older browsers and some data stores don't support CORS, and code that runs in these browsers might be able to use a valet key to provide access to data in a different domain, such as a cloud storage account.

## When to use this pattern

This pattern is useful for the following situations:

- To minimize resource loading and maximize performance and scalability. Using a valet key doesn't require the resource to be locked, no remote server call is required, there's no limit on the number of valet keys that can be issued, and it avoids a single point of failure resulting from performing the data transfer through the application code. Creating a valet key is typically a simple cryptographic operation of signing a string with a key.

- To minimize operational cost. Enabling direct access to stores and queues is resource and cost efficient, can result in fewer network round trips, and might allow for a reduction in the number of compute resources required.

- When clients regularly upload or download data, particularly where there's a large volume or when each operation involves large files.

- When the application has limited compute resources available, either due to hosting limitations or cost considerations. In this scenario, the pattern is even more helpful if there are many concurrent data uploads or downloads because it relieves the application from handling the data transfer.

- When the data is stored in a remote data store or a different datacenter. If the application was required to act as a gatekeeper, there might be a charge for the additional bandwidth of transferring the data between datacenters, or across public or private networks between the client and the application, and then between the application and the data store.

This pattern might not be useful in the following situations:

- If the application must perform some task on the data before it's stored or before it's sent to the client. For example, if the application needs to perform validation, log access success, or execute a transformation on the data. However, some data stores and clients are able to negotiate and carry out simple transformations such as compression and decompression (for example, a web browser can usually handle GZip formats).

- If the design of an existing application makes it difficult to incorporate the pattern. Using this pattern typically requires a different architectural approach for delivering and receiving data.

- If it's necessary to maintain audit trails or control the number of times a data transfer operation is executed, and the valet key mechanism in use doesn't support notifications that the server can use to manage these operations.

- If it's necessary to limit the size of the data, especially during upload operations. The only solution to this is for the application to check the data size after the operation is complete, or check the size of uploads after a specified period or on a scheduled basis.

## Example

Azure supports shared access signatures on Azure Storage for granular access control to data in blobs, tables, and queues, and for Service Bus queues and topics. A shared access signature token can be configured to provide specific access rights such as read, write, update, and delete to a specific table; a key range within a table; a queue; a blob; or a blob container. The validity can be a specified time period or with no time limit.

Azure shared access signatures also support server-stored access policies that can be associated with a specific resource such as a table or blob. This feature provides additional control and flexibility compared to application-generated shared access signature tokens, and should be used whenever possible. Settings defined in a server-stored policy can be changed and are reflected in the token without requiring a new token to be issued, but settings defined in the token can't be changed without issuing a new token. This approach also makes it possible to revoke a valid shared access signature token before it's expired.

> For more information see [Introducing Table SAS (Shared Access Signature), Queue SAS and update to Blob SAS](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/) and [Using Shared Access Signatures](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/) on MSDN.

The following code shows how to create a shared access signature token that's valid for five minutes. The `GetSharedAccessReferenceForUpload` method returns a shared access signatures token that can be used to upload a file to Azure Blob Storage.

```csharp
public class ValuesController : ApiController
{
  private readonly CloudStorageAccount account;
  private readonly string blobContainer;
  ...
  /// <summary>
  /// Return a limited access key that allows the caller to upload a file
  /// to this specific destination for a defined period of time.
  /// </summary>
  private StorageEntitySas GetSharedAccessReferenceForUpload(string blobName)
  {
    var blobClient = this.account.CreateCloudBlobClient();
    var container = blobClient.GetContainerReference(this.blobContainer);

    var blob = container.GetBlockBlobReference(blobName);

    var policy = new SharedAccessBlobPolicy
    {
      Permissions = SharedAccessBlobPermissions.Write,

      // Specify a start time five minutes earlier to allow for client clock skew.
      SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5),

      // Specify a validity period of five minutes starting from now.
      SharedAccessExpiryTime = DateTime.UtcNow.AddMinutes(5)
    };

    // Create the signature.
    var sas = blob.GetSharedAccessSignature(policy);

    return new StorageEntitySas
    {
      BlobUri = blob.Uri,
      Credentials = sas,
      Name = blobName
    };
  }

  public struct StorageEntitySas
  {
    public string Credentials;
    public Uri BlobUri;
    public string Name;
  }
}
```

> The complete sample is available in the ValetKey solution available for download from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key). The ValetKey.Web project in this solution contains a web application that includes the `ValuesController` class shown above. A sample client application that uses this web application to retrieve a shared access signatures key and upload a file to blob storage is available in the ValetKey.Client project.

## Next steps

The following patterns and guidance might also be relevant when implementing this pattern:
- A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key).
- [Gatekeeper pattern](gatekeeper.md). This pattern can be used in conjunction with the Valet Key pattern to protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service. The gatekeeper validates and sanitizes requests, and passes requests and data between the client and the application. Can provide an additional layer of security, and reduce the attack surface of the system.
- [Static Content Hosting pattern](static-content-hosting.md). Describes how to deploy static resources to a cloud-based storage service that can deliver these resources directly to the client to reduce the requirement for expensive compute instances. Where the resources aren't intended to be publicly available, the Valet Key pattern can be used to secure them.
- [Introducing Table SAS (Shared Access Signature), Queue SAS and update to Blob SAS](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/)
- [Using Shared Access Signatures](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)
- [Shared Access Signature Authentication with Service Bus](https://azure.microsoft.com/documentation/articles/service-bus-shared-access-signature-authentication/)
