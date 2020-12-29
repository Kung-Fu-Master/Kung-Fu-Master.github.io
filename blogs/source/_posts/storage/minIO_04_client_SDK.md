---
title: MinIO 04 client SDK
tags: storage
categories:
- storage
- minio
---

## Reference

参考下载并生成AWS s3 SDK
[Setting Up the AWS SDK for C++](https://docs.aws.amazon.com/sdk-for-cpp/v1/developer-guide/setup.html)

[source code](https://github.com/aws/aws-sdk-cpp)
[SDK_usage_guide.md](https://github.com/aws/aws-sdk-cpp/blob/master/Docs/SDK_usage_guide.md)
[Using CMake Exports with the AWS SDK for C++](https://aws.amazon.com/blogs/developer/using-cmake-exports-with-the-aws-sdk-for-c/)
[AWS SDK for C++ Developer Guide](https://docs.aws.amazon.com/sdk-for-cpp/v1/developer-guide/welcome.html)
[Creating, Listing, and Deleting Buckets](https://docs.aws.amazon.com/sdk-for-cpp/v1/developer-guide/examples-s3-buckets.html)
[AWS Code Sample Catalog](https://docs.aws.amazon.com/code-samples/latest/catalog/welcome.html)
[aws-doc-sdk-examples](https://github.com/awsdocs/aws-doc-sdk-examples/tree/master/cpp/example_code/s3)

## **Samples**

### **c++ SDK**

gcc/g++ 4.9+
	$ g++ -v
	......
	gcc version 8.3.1 20190311 (Red Hat 8.3.1-3) (GCC)

安装cmake

	$ cmake --version
	cmake version 3.17.0
	CMake suite maintained and supported by Kitware (kitware.com/cmake).
安装依赖库openssl & curl

	$ yum install openssl-devel libcurl-devel
升级curl

	$ rpm -Uvh https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libmetalink-0.1.3-1.el7.x86_64.rpm
	$ yum --showduplicates list curl --disablerepo="*" --enablerepo="city*"
	$ curl.x86_64                                            7.72.0-2.0.cf.rhel7                                             @city-fan.org
	......
	Available Packages
	curl.x86_64                                            7.72.0-2.0.cf.rhel7                                             city-fan.org

#### Download aws sdk

	$ wget https://github.com/aws/aws-sdk-cpp/archive/1.8.55.tar.gz
	$ tar -zxvf 1.8.55.tar.gz
	$ mkdir build_dir
	$ cd build_dir
	(编译全部, 会耗费一个小时左右时间, 推荐使用以下只编译S3 service)$ cmake /home/***/aws-sdk-cpp-1.8.55/ -DCMAKE_BUILD_TYPE=Debug
	$ cmake /home/***/aws-sdk-cpp-1.8.55/ -D CMAKE_BUILD_TYPE=[Debug | Release] -D BUILD_ONLY="s3"
	make
	make install // 安装头文件等到系统/usr/local/include目录

#### 开发SDK
拷贝aws sdk的头文件和动态库到项目目录

	// 拷贝整个aws头文件到项目目录
	cp /usr/local/include/aws ***/cpp/include/
	
	// 拷贝aws两个动态库到项目目录
	cp /usr/local/lib64/libaws-cpp-sdk-core.so ***/cpp/libs/
	cp /usr/local/lib64/libaws-cpp-sdk-s3.so ***/cpp/libs/

查看项目目录
	$ ls cpp
	CMakeLists.txt  include/  libs/  main/  src/

	$ tree cpp
	cpp
	├── CMakeLists.txt
	├── include
	│   ├── aws
	│   │   ├── core
	│   │   │   ├── Aws.h
	│   │   │   └── ....h
	│   │   ├── s3
	│   │   │   ├── model
	│   │   │   │   ├── Bucket.h
	│   │   │   │   ├── CreateBucketRequest.h
	│   │   │   │   └── ....h
	│   │   │   ├── S3Client.h
	│   │   │   └── ....h
	│   │   └── utils
	│   │       ├── StringUtils.h
	│   │       ├── UUID.h
	│   │       └── ....h
	│   └── minio_client_sdk.h
	├── libs
	│   ├── libaws-cpp-sdk-core.so
	│   ├── libaws-cpp-sdk-s3.so
	│   └── libminio_client_SDK.so
	├── main
	│   ├── CMakeLists.txt
	│   └── main.cpp
	└── src
	    ├── CMakeLists.txt
	    └── minio_client_sdk.cpp
#### **cpp/CMakeLists.txt**

	CMAKE_MINIMUM_REQUIRED(VERSION 3.14)
	
	project(minio_SDK)
	
	#头文件查找优先级高于系统默认目录/usr/include和/usr/local/include
	INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/include)
	
	ADD_SUBDIRECTORY(${PROJECT_SOURCE_DIR}/src)
	
	add_subdirectory(${PROJECT_SOURCE_DIR}/main)
	
	#MESSAGE(STATUS ${SRC_LIST})

#### **cpp/main**

**cpp/main/CMakeLists.txt**

	#头文件搜索目录,PROJECT_SOURCE_DIR为cmake预定义变量，项目的顶层目录
	include_directories(${PROJECT_SOURCE_DIR}/include)
	
	#链接库搜索目录
	#它相当于g++命令的-L选项的作用，也相当于环境变量中增加LD_LIBRARY_PATH的路径的作用
	link_directories(${PROJECT_SOURCE_DIR}/libs)
	
	#重新定义目标二进制可执行文件的存放位置
	set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/build/bin)
	
	add_executable(main main.cpp) #生成.cpp文件的可执行文件
	
	#指定多个链接库
	target_link_libraries(main minio_client_SDK aws-cpp-sdk-core aws-cpp-sdk-s3)

**cpp/main/main.cpp**

	#include <iostream>
	#include <aws/s3/S3Client.h>
	#include <minio_client_sdk.h>
	using namespace std;
	
	int main(int argc, char *argv[])
	{
	    string minio_server_address = "127.0.0.1:30007";
	    string access_key = "hceminio";
	    string access_secret = "hceminio123";
	    Aws::SDKOptions m_options;
	    S3Client *m_client = access_minio_server(minio_server_address, m_options, access_key, access_secret);
	
	    string uuid = random_UUID();
	    cout << "UUID: " << uuid << endl;
	    delete_Bucket(m_client, "bucket04");
	    list_Buckets(m_client);
	    create_Bucket(m_client, "bucket04");
	
	    list_Objects(m_client, "bucket01");
	    delete_Object(m_client, "bucket01", "pic_object1");
	    download_Object(m_client, "bucket01", "clean.sh", "t1.txt");
	    upload_Object(m_client, "bucket01", "pic_object4", "tt.txt");
	
	    close_minio_server(m_client, m_options);
	
	    return 0;
	}

#### **cpp/src**
**cpp/src/CMakeLists.txt**

	#指定头文件搜索目录，若顶层CMakeLists.txt也指定了文件搜索目录，则该处可以省略
	#头文件查找优先级高于系统默认目录/usr/include和/usr/local/include
	include_directories(${PROJECT_SOURCE_DIR}/include)
	
	FILE(GLOB SRC_LIST "${PROJECT_SOURCE_DIR}/src/*.cpp")
	
	#重新定义目标链接库文件的存放位置
	SET(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/libs)
	
	#将SRC_LIST变量中的所有.cpp文件编译生成库名为add的静态库
	#SHARED为生成动态库, STATIC为生成静态库.
	ADD_LIBRARY(minio_client_SDK SHARED ${SRC_LIST})

**cpp/src/minio_client_sdk.cpp**

	#define MINIO_CLIENT_SDK
	#include <iostream>
	#include <fstream>
	
	#include <aws/s3/S3Client.h>
	#include <aws/core/Aws.h>
	#include <aws/core/auth/AWSCredentialsProvider.h>
	#include <aws/core/utils/StringUtils.h>
	#include <aws/core/utils/UUID.h>
	
	using namespace std;
	using namespace Aws::S3;
	using namespace Aws::S3::Model;
	
	#include <aws/s3/model/Bucket.h>
	#include <aws/s3/model/CreateBucketRequest.h>
	#include <aws/s3/model/DeleteBucketRequest.h>
	#include <aws/s3/model/DeleteObjectRequest.h>
	#include <aws/s3/model/GetObjectRequest.h>
	#include <aws/s3/model/ListObjectsRequest.h>
	#include <aws/s3/model/Object.h>
	#include <aws/s3/model/PutObjectRequest.h>
	
	S3Client *access_minio_server(string minio_server_address, Aws::SDKOptions m_options, string access_key, string access_secret)
	{
	    S3Client *m_client = {NULL};
	
	    Aws::InitAPI(m_options);
	    Aws::Client::ClientConfiguration cfg;
	    cfg.endpointOverride = minio_server_address.c_str();
	    cfg.scheme = Aws::Http::Scheme::HTTP;
	    cfg.verifySSL = false;
	    m_client = new S3Client(Aws::Auth::AWSCredentials(access_key.c_str(), access_secret.c_str()), cfg, Aws::Client::AWSAuthV4Signer::PayloadSigningPolicy::Never, false, Aws::S3::US_EAST_1_REGIONAL_ENDPOINT_OPTION::NOT_SET);
	    return m_client;
	}
	
	bool close_minio_server(S3Client *m_client, Aws::SDKOptions m_options)
	{
	    if (m_client != nullptr)
	    {
	        delete m_client;
	        m_client = NULL;
	    }
	    Aws::ShutdownAPI(m_options);
	
	    return true;
	}
	
	string random_UUID()
	{
	    Aws::String random_uuid = Aws::Utils::UUID::RandomUUID();
	    Aws::String uuid = Aws::Utils::StringUtils::ToLower(random_uuid.c_str());
	    return uuid.c_str();
	}
	
	bool list_Buckets(S3Client *m_client)
	{
	    Aws::S3::Model::ListBucketsOutcome outcome = m_client->ListBuckets();
	    if (outcome.IsSuccess())
	    {
	        std::cout << "Bucket names:" << std::endl
	                  << std::endl;
	        Aws::Vector<Aws::S3::Model::Bucket> buckets = outcome.GetResult().GetBuckets();
	        for (Aws::S3::Model::Bucket &bucket : buckets)
	        {
	            std::cout << bucket.GetName() << std::endl;
	        }
	        return true;
	    }
	    else
	    {
	        std::cout << "Error: ListBuckets: " << outcome.GetError().GetMessage() << std::endl;
	
	        return false;
	    }
	}
	
	bool create_Bucket(S3Client *m_client, string bucketName)
	{
	    Aws::S3::Model::CreateBucketRequest request;
	    request.SetBucket(bucketName.c_str());
	    Aws::S3::Model::CreateBucketOutcome outcome = m_client->CreateBucket(request);
	    if (!outcome.IsSuccess())
	    {
	        auto err = outcome.GetError();
	        std::cout << "Error: CreateBucket: " << err.GetExceptionName() << ": " << err.GetMessage() << std::endl;
	        return false;
	    }
	    return true;
	}
	
	bool delete_Bucket(S3Client *m_client, string bucketName)
	{
	    Aws::S3::Model::DeleteBucketRequest request;
	    request.SetBucket(bucketName.c_str());
	    Aws::S3::Model::DeleteBucketOutcome outcome = m_client->DeleteBucket(request);
	    if (!outcome.IsSuccess())
	    {
	        auto err = outcome.GetError();
	        std::cout << "Error: DeleteBucket: " << err.GetExceptionName() << ": " << err.GetMessage() << std::endl;
	        return false;
	    }
	    return true;
	}
	
	bool list_Objects(S3Client *m_client, string bucketName)
	{
	    Aws::S3::Model::ListObjectsRequest request;
	    request.WithBucket(bucketName.c_str());
	    auto outcome = m_client->ListObjects(request);
	    if (outcome.IsSuccess())
	    {
	        std::cout << "Objects in bucket '" << bucketName << "':" << std::endl
	                  << std::endl;
	        Aws::Vector<Aws::S3::Model::Object> objects = outcome.GetResult().GetContents();
	        for (Aws::S3::Model::Object &object : objects)
	        {
	            std::cout << object.GetKey() << std::endl;
	        }
	        return true;
	    }
	    else
	    {
	        std::cout << "Error: ListObjects: " << outcome.GetError().GetMessage() << std::endl;
	        return false;
	    }
	}
	
	bool download_Object(S3Client *m_client, string bucketName, string objectKey, string pathKey)
	{
	    Aws::S3::Model::GetObjectRequest object_request;
	    object_request.WithBucket(bucketName.c_str()).WithKey(objectKey.c_str());
	    auto get_object_outcome = m_client->GetObject(object_request);
	    if (get_object_outcome.IsSuccess())
	    {
	        Aws::OFStream local_file;
	        local_file.open(pathKey.c_str(), std::ios::out | std::ios::binary);
	        local_file << get_object_outcome.GetResult().GetBody().rdbuf();
	        std::cout << "Done!" << std::endl;
	        return true;
	    }
	    else
	    {
	        std::cout << "GetObject error: " << get_object_outcome.GetError().GetExceptionName() << " " << get_object_outcome.GetError().GetMessage() << std::endl;
	        return false;
	    }
	}
	
	bool upload_Object(S3Client *m_client, string bucketName, string objectKey, string pathKey)
	{
	    PutObjectRequest putObjectRequest;
	    putObjectRequest.WithBucket(bucketName.c_str()).WithKey(objectKey.c_str());
	    auto input_data = Aws::MakeShared<Aws::FStream>("PutObjectInputStream", pathKey.c_str(), std::ios_base::in | std::ios_base::binary);
	    putObjectRequest.SetBody(input_data);
	    auto putObjectResult = m_client->PutObject(putObjectRequest);
	    if (putObjectResult.IsSuccess())
	    {
	        std::cout << "Done!" << std::endl;
	        return true;
	    }
	    else
	    {
	        std::cout << "PutObject error: " << putObjectResult.GetError().GetExceptionName() << " " << putObjectResult.GetError().GetMessage() << std::endl;
	        return false;
	    }
	}
	
	bool delete_Object(S3Client *m_client, string bucketName, string objectKey)
	{
	    Aws::S3::Model::DeleteObjectRequest request;
	    request.WithKey(objectKey.c_str()).WithBucket(bucketName.c_str());
	    Aws::S3::Model::DeleteObjectOutcome outcome = m_client->DeleteObject(request);
	    if (!outcome.IsSuccess())
	    {
	        auto err = outcome.GetError();
	        std::cout << "Error: DeleteObject: " << err.GetExceptionName() << ": " << err.GetMessage() << std::endl;
	        return false;
	    }
	    else
	    {
	        return true;
	    }
	}

#### **cpp/include**
**cpp/include/minio_client_sdk.h**

	#include <iostream>
	#include <aws/core/Aws.h>
	#include <aws/s3/S3Client.h>
	
	using namespace std;
	using namespace Aws::S3;
	
	S3Client *access_minio_server(string minio_server_address, Aws::SDKOptions m_options, string access_key, string access_secret);
	bool close_minio_server(S3Client *m_client, Aws::SDKOptions m_options);
	
	string random_UUID();
	bool list_Buckets(S3Client *m_client);
	bool create_Bucket(S3Client *m_client, string bucketName);
	bool delete_Bucket(S3Client *m_client, string bucketName);
	
	bool list_Objects(S3Client *m_client, string bucketName);
	bool download_Object(S3Client *m_client, string bucketName, string objectKey, string pathKey);
	bool upload_Object(S3Client *m_client, string bucketName, string objectKey, string pathKey);
	bool delete_Object(S3Client *m_client, string bucketName, string objectKey);

#### **编译执行**

	$ cd cpp
	$ mkdir build
	$ cd build
	$ cmake ..
	$ make
	$ ./bin/main



