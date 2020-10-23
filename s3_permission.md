# 해결해야할 일.
> S3를 쓰는데 정작 s3에 업로드하는 것보다 권한 관리가 더 어렵다. 권한 설정하는게 한가지 방법이 아니고 여러가지가 있다. 모두 대략 파악해보고 내가 쓸 방법은 깊이 파악해보자.

# S3 권한 관리 종류
- Public Access
- ACL (사용자, 버킷, 객체)
- Bucket Policy

<br>

![Public Access](https://raw.githubusercontent.com/moongq/writings/master/images/pubilc_access_list.PNG)

### Block all public access
- Block public access to buckets and objects granted through `new` access control lists (ACLs)<br>
새로운 ACL을 통해 퍼블릭 접근이 가능해진 버켓과 객체들에 퍼블릭 접근을 제한한다.

Block public access to buckets and objects granted through any access control lists (ACLs)
S3 will ignore all ACLs that grant public access to buckets and objects.
어떠한 ACL을 통해 퍼블릭 접근이 가능해진 버켓과 객체들에 퍼블릭 접근을 제한한다.

Block public access to buckets and objects granted through new public bucket or access point policies

새로운 퍼블릭 버켓, 혹은 접근 정책에 의해 부여된 퍼블릭 접근을 제한한다.


S3 will block new bucket and access point policies that grant public access to buckets and objects. This setting doesn't change any existing policies that allow public access to S3 resources.
Block public and cross-account access to buckets and objects through any public bucket or access point policies



S3 will ignore public and cross-account access for buckets or access points with policies that grant public access to buckets and objects.