# 다른 계정의 서울 Region S3로 이사가기

AWS 프리티어 기간 만료가 가까워서 새 계정을 개설해서 옮겨보려고 한다.
마침 AWS에 서울 리전(Region)이 개설되었고, S3도 [개설 서비스 목록](http://www.awsseoul.kr/sub01.html#introseoul02)에 포함되어 있다.

![](http://i.imgur.com/4mcAI8G.png)

새 계정의 S3로 이사가는 대략적인 큰 순서는 다음과 같다.

1. AWS 새 계정 생성, IAM Group 생성, IAM 계정 생성, ACCESS_KEY 발급
2. credentials 파일에 새 IAM 계정 정보 추가
3. config 파일에 profile 정보 추가
4. 새 계정의 S3에 버킷 생성
5. 기존 S3 버킷 권한 수정
6. 새 계정의 Custom Policy 설정
7. AWS CLI 설치
8. aws sync 명령 실행
9. Application 설정

이 중에서 1번은 AWS 사용자라면 다들 알만한 내용이니 과감히 뛰어넘고 2번부터 정주행하자.

# credentials 파일에 새 IAM 계정 정보 추가

보통 ~/.aws/ 에 위치한 credentials 파일에 새 IAM 계정의 ACCESS_KEY 정보를 아래와 같이 `[프로파일이름]`의 형식으로 프로파일 이름을 지정해서 추가한다.

기존 계정의 ACCESS_KEY 정보에는 일단 default 라는 프로파일 이름을 지정한다.

![](http://i.imgur.com/ASkOyEe.png)

credentials 관련 정보는 http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-multiple-profiles 를 참고한다.

# config 파일에 profile 정보 추가

~/.aws/config 파일에 아래와 같이 Region과 출력 형식(output)을 지정해준다. 
서울은 ap-northeast-2 이며, 출력 형식은 json과 text가 가능한데 CLI에서는 text가 아무래도 익숙하다.

Region에 지정할 값은 http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region 를 참고한다.

 ![](http://i.imgur.com/fYcSdoT.png)

config 관련 정보도 http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-multiple-profiles 를 참고한다.

# 새 계정의 S3에 버킷 생성

아래와 같이 Bucket 이름을 지정하고, Region에 Seoul을 지정하고 Create를 클릭

![](http://i.imgur.com/DNQpVSN.png)

# 기존 S3 버킷 권한 수정

기존 S3 버킷에서 새 계정의 S3 버킷으로 복사하려면, 새 계정으로 기존의 S3 버킷에 접근할 수 있어야 한다.  

## 새 계정의 정보 파악

기존 S3 버킷에 새 계정으로 접근할 수 있도록 설정하려면 먼저 새 계정의 `account number`와 `account_name` 정보를 파악해둬야 한다.
 
- 해당 정보는 아래와 같이 새로 생성한 계정의 AWS Console > Security & Credentials >  Continue to Security Credentials > Users 에서 새로 생성한 계정을 선택하면 확인할 수 있다.
    
    ![](http://i.imgur.com/LPe9g4y.png)
    
    ![](http://i.imgur.com/PtyJSOM.png)

    ![](http://i.imgur.com/j3MJJXZ.png)

## 기존 S3 버킷의 Policy 수정

기존 S3 버킷의 Permissions > Bucket Policy Editor 에 아래의 내용을 입력한다.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DelegateS3Access",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::ACCOUNT_NUMBER:user/NEW_ACCOUNT_NAME"
            },
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::기존.버킷.이름",
                "arn:aws:s3:::기존.버킷.이름/*"
            ]
        }
    ]
}
```
- `ACCOUNT_NUMBER`와 `NEW_ACCOUNT_NAME`에는 앞에서 파악한 **새 계정의 정보**를 입력한다. 
- `Action` 항목에는 현재는 편의상 s3의 모든 권한을 부여했는데, **실무에서는 필요한 권한만 부여하는 것이 안전**하다.
- `기존.버킷.이름`에는 말 그대로 복사할 때 source가 되는 기존 버킷 이름을 기술한다. 

# 새 계정의 Custom Policy 설정

이제 새 계정이 기존 계정의 S3 버킷에 접근할 수 있도록 계정의 Policy를 설정해줘야 한다.
IAM Group에 Policy를 설정할 수도 있고, IAM User에 Policy를 설정할 수도 있는데, 여기서는 User에 설정하는 방식을 설명한다.

- 새로 생성한 계정의 AWS Console > Security & Credentials > Continue to Security Credentials > Users 에서 새로 생성한 계정을 선택하고,    

- 하단의 Permisions 탭 > Inline Policies > (최초 생성일 경우는 아래와 같이 click here를 클릭, 이미 추가된 Policy가 있는 경우 Create User Policy를 클릭) > Custom Policy > Select를 클릭해서,

    ![](http://i.imgur.com/aRowuEu.png)

    - Policy Name : GetObjectsFromOldGradnetS3 와 같이 적당한 이름을 지정
    - Policy Document : 아래의 Policy 입력

        ```
        {
            "Version": "2012-10-17",
            "Statement": {
                "Effect": "Allow",
                "Action": "s3:*",
                "Resource": [
                    "arn:aws:s3:::기존.버킷.이름",
                    "arn:aws:s3:::기존.버킷.이름/*",
                    "arn:aws:s3:::새.버킷.이름",
                    "arn:aws:s3:::새.버킷.이름/*"                    
                ]
            }
        }
        ```
    - 입력 화면 예시        

        ![](http://i.imgur.com/Syc5gGp.png)

    - Validate Policy 를 클릭해서 입력 내용을 검증하고, Apply Policy를 클릭해서 Policy를 등록하면 아래와 같이 Policy가 추가된다.

        ![](http://i.imgur.com/ZdkINg5.png)

이걸로 AWS 콘솔에서 준비할 일은 끝이다. 이제 실제 복사 명령을 내려보자.

# AWS CLI 설치

일반적인 리눅스라면 `sudo pip install awscli` 라는 명령으로 손쉽게 설치할 수 있지만, 만약 이 명령이 안 먹는다면 python도 깔아야 하고 pip도 깔아야 한다. 

AWS CLI 설치를 위한 정보는 http://docs.aws.amazon.com/ko_kr/cli/latest/userguide/installing.html 를 참고한다.

# aws sync 명령 실행

복사 하나 하는게 뭐 이리 힘든지.. 드디어 마지막이다. AWS 문서를 샅샅이 뒤져야 찾을 수 있었던..

이 글의 토대가 된 https://aws.amazon.com/ko/premiumsupport/knowledge-center/account-transfer-s3/ 에 마땅히 있어야 할 내용이고, 있었다면 이런 글 쓸 필요도 없었겠지만..

먼저 터미널에서 아래의 명령을 실행해서 source인 기존 S3 버킷의 Region을 파악한다. 

>aws s3api get-bucket-location --bucket 기존.버킷.이름

![](http://i.imgur.com/NJ9mnZg.png)

내 경우 기존의 S3 버킷이 Tokyo에 있었으므로 ap-northeast-1이라는 값이 결과로 나온다.

이번엔 target이 될 새 S3 버킷의 Region을 파악한다. 주의할 점은 새 S3 버킷은 새로 생성한 계정으로만 접근할 수 있으므로, `--profile` 옵션으로 프로파일을 지정해줘야 한다. 
결과는 물론 Seoul이니까 ap-northeast-2가 나올 것이다.

>aws s3api get-bucket-location --bucket 새.버킷.이름 --profile 프로파일이름

![](http://i.imgur.com/jps84sj.png)

자 이제 마지막 복사 명령만 남았다. AWS 문서에는 달랑 아래의 명령을 실행하면 된다고 나와있지만,
>aws s3 sync s3://sourcebucket s3://destinationbucket

이건 source와 target이 동일한 Region 내에 있을 때만, 그리고 새 계정이 AWS CLI의 default 프로파일로 설정되어 있을 때나 쓸 수 있는 특별한 사례에 불과하다.

내 경우처럼 기존 계정이 default로 되어 있지 않아 프로파일을 지정해줘야 하고, source 버킷과 target 버킷의 Region이 다르고, 접근 권한을 변경해줘야 할 때도 쓸 수 있는 일반적인 해법이 필요하다. 더 많은 옵션이 필요할 수도 있지만 일단 다음의 명령으로 해결했다. 

>aws s3 sync s3://기존.버킷.이름 s3://새.버킷.이름 --source-region 기존버킷의Region --region 새버킷의Region --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers full=id=새계정의CanonicalID --storage-class REDUCED_REDUNDANCY --profile onetouch

![](http://i.imgur.com/PuSGBtm.png)

필요한 옵션은 아래와 같다.

- --source-region : source 버킷의 Region 정보
- --region : target 버킷의 Region 정보
- --grants : 복사되는 자원에 대한 접근 권한 설정
    - `Permission=Grantee_type=Grantee_ID [Permission=Grantee_type=Grantee_ID ...]`의 형태로 여러 권한을 한 번에 설정할 수 있다.
        - Permission은 `read`, `readacl`, `writeacl`, `full` 중 하나로 설정 가능
        - Grantee_type은 `uri`, `emailaddress`, `id` 중 하나로 설정 가능
        - Grantee_ID는 Grantee_type의 설정 내용에 따라 달라지는데, Grantee_type이 `uri` 인 경우 http://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/acl-overview.html#canned-acl 를 참고, `emailaddress`인 경우 이메일주소, `id`인 경우 계정의 Canonical ID를 입력한다.
            - Canonical ID는 AWS 콘솔 > Security Credentials > Continue to Security Credentials > Account Identifiers 를 클릭하면 확인할 수 있다.
- --storage-class와 --profile은 설명 생략~

더 자세한 옵션 설정이 필요하다면, http://docs.aws.amazon.com/cli/latest/reference/s3/sync.html 를 참고한다.

여기까지 하면 기존 S3 버킷에 있던 내용물의 복사는 완료된다. 하지만 실제 Application에서 접근해서 사용하기 위해서는 할 일이 더 남아있다.

# Application 설정

구체적인 설정 내용은 Spring을 기준으로 한다.

## 계정 정보 업데이트

먼저 새 계정으로 새 S3 버킷에 접근하도록 계정 정보를 업데이트 해줘야 한다. 

단순하게는 ~/.aws/credentials 파일에서 새 계정의 ACCESS_KEY 정보 위에 [default]를 지정해서 기본 프로파일로 지정해주면 된다.

아니면 프로파일을 명시적으로 지정해주는 방식을 쓸 수도 있는데, AmazonS3Client를 아래와 같이 설정해준다.

```xml
<bean id="awsCredentialsProvider" class="com.amazonaws.auth.profile.ProfileCredentialsProvider">
	<constructor-arg value="#{app['aws.credential.profile']}"/>
</bean>

<bean id="s3Client" class="com.amazonaws.services.s3.AmazonS3Client">
	<constructor-arg ref="awsCredentialsProvider"/>
</bean>
```
`ProfileCredentialsProvider`의 생성자 인자로 들어갈 값은 ~/.aws/credentials나 ~/.aws/config에서 지정한 프로파일 이름이다. 읽어오는 방식은 Application의 property 사용 방식을 따르면 된다.

## S3 Endpoint 지정

[서울 Region에서 서비스하는 S3는 Signature Version 4만 지원한다](http://docs.aws.amazon.com/ko_kr/general/latest/gr/rande.html#s3_region). 
따라서 AmazonS3Client 생성 시 Endpoint나 Region을 지정해주지 않으면, Signature Version 2를 지원하는 S3에서는 볼 수 없었던 아래와 같은 경고 메시지가 나온다.

![](http://i.imgur.com/vPbr3LI.png)

이를 해결하려면 Enpoint나 Region을 지정해줘야 하는데, 아무래도 텍스트로 지정할 수 있는 Endpoint를 지정해주는 것이 더 편리하다.

앞에서 AmazonS3Client를 설정한 부분에 아래와 같이 endpoint라는 속성 값을 추가로 지정해준다. 서울 Region의 경우 실제 들어갈 문자열은 `s3.ap-northeast-2.amazonaws.com`이다. 

```xml
<bean id="awsCredentialsProvider" class="com.amazonaws.auth.profile.ProfileCredentialsProvider">
	<constructor-arg value="#{app['aws.credential.profile']}"/>
</bean>

<bean id="s3Client" class="com.amazonaws.services.s3.AmazonS3Client">
	<constructor-arg ref="awsCredentialsProvider"/>
	<property name="endpoint" value="#{app['s3.endpoint']}" />
</bean>
```

# 정리

>- AWS가 서울 Region을 개설했다.

>- S3를 서울 Region으로 이사시켜보자.

>- 계정까지 새로 만든다면 S3를 이사하는게 꽤 복잡하다.
>    1. AWS 새 계정 생성, IAM Group 생성, IAM 계정 생성, ACCESS_KEY 발급
>    2. credentials 파일에 새 IAM 계정 정보 추가
>    3. config 파일에 profile 정보 추가
>    4. 새 계정의 S3에 버킷 생성
>    5. 기존 S3 버킷 권한 수정
>    6. 새 계정의 Custom Policy 설정
>    7. AWS CLI 설치
>    8. aws sync 명령 실행
>    9. Application 설정

>- 복잡하지만 약간이나마 가격도 싸고 속도도 좋다고 하니 이 글 참고 삼아서 옮겨보자.

# 참고 자료

- https://aws.amazon.com/ko/premiumsupport/knowledge-center/account-transfer-s3/
- http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-multiple-profiles
- http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region
- http://docs.aws.amazon.com/ko_kr/cli/latest/userguide/installing.html
- http://docs.aws.amazon.com/cli/latest/reference/s3/sync.html
- http://docs.aws.amazon.com/ko_kr/general/latest/gr/rande.html#s3_region