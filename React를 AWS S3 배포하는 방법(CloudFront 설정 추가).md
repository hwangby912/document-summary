# React를 AWS S3 배포하는 방법(CloudFront 설정 추가)

1. **<u>Build</u>**

   ```react
   npx create-react-app projectName
   cd projectName
   
   yarn build
   ```

2. **<u>AWS에서 IAM 권한 설정</u>** 

   1. 서비스 탭에 들어가서 IAM 검색 후 들어가서 사용자 탭 누르기
      ![AWS IAM 사용자 tab](C:\Users\HBY\Desktop\쓸만한 것들\국비에서 한 것들\시큐어 코딩 기반 블록체인 개발\AWS 관련 정리자료\AWS IAM 사용자 tab.png)

   2. 사용자가 없다면 추가하기(그 후 모두 다음다음을 누르기)
      ![AWS IAM 사용자 만들기 1](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS IAM 사용자 만들기 1.png)
   3. 사용자 추가가 완료 되었고 이 Page가 중요하며 다시는 볼 수 없기 때문에 <u>**Access Key와 Secret Access Key를 반드시 저장**</u>하자
      ![AWS IAM 사용자 만들기 2](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS IAM 사용자 만들기 2.png)
      저는 이미 있기에 기존에 있는 사용자로 진행함을 알립니다. 

3. **<u>AWS CLI 설치</u>**

   설치를 완료하였다면 cmd로 들어가서 다음과 같이 입력해줍니다. 
   여기서 2-3에 저장한 Access Key와 Secret Access Key를 사용해야합니다. 
   region name은 https://docs.aws.amazon.com/ko_kr/general/latest/gr/rande.html 에서 **<u>Amazon API Gateway 데이터 영역</u>**을 보고 선택해주도록 한다. 

   ```
   aws configure --profile username
   AWS Access Key ID [None]: USER_AWS_ACCESS_KEY
   AWS Secret Access Key [None]: USER_AWS_SECRET_ACCESS_KEY
   Default region name [None]: ap-northeast-2
   Default output format [None]: json
   ```

4. **<u>S3 설정한 뒤 Bucket 생성</u>**

   1. 서비스 탭에 들어가서 S3 검색 후 들어가기
      ![AWS S3 Bucket](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS S3 Bucket.png)

   2. 버킷 만들기 버튼 누른 후 이름 및 리전 부분에서는 **<u>버킷 이름</u>**만 입력하기
      ![AWS S3 Bucket 만들기 1](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS S3 Bucket 만들기 1.png)

   3. 옵션 구성 부분은 아무것도 하지 않고 다음 버튼 클릭
      ![AWS S3 Bucket 만들기 2](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS S3 Bucket 만들기 2.png)

   4. 권한 설정 부분은 **<u>모든 퍼블릭 엑세스 차단</u>** 부분 **<u>체크박스 해제</u>** 후 다음 버튼 클릭

      ![AWS S3 Bucket 만들기 3](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS S3 Bucket 만들기 3.png)

   5. 버킷 만들기 클릭
      ![AWS S3 Bucket Create Result](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS S3 Bucket Create Result.png)

5. S3에 익명 사용자에게 읽기전용 권한 부여하기

   1. 만든 버킷인 user-bucketname 클릭 후 속성 탭 누르기 

      ![AWS S3 Bucket 속성](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS S3 Bucket 속성.png)

   2. 3번째에 보이는 정적 웹 사이트 호스팅 클릭 후 속성 탭에 들어 간 뒤 같이 입력해주기
      ![AWS S3 Bucket 정적 웹 사이트 호스팅](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS S3 Bucket 정적 웹 사이트 호스팅.png)

   3. 정적 웹 사이트 호스팅의 권한 탭 클릭하고 버킷 정책 클릭하기

      ```json
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "AddPerm",
                  "Effect": "Allow",
                  "Principal": "*",
                  "Action": "s3:GetObject",
                  "Resource": "arn:aws:s3:::user-bucketname"
              }
          ]
      }
      ```

      Resource 부분의 user-bucketname만 맞춰서 변경하고 나머지는 그대로 써준 뒤, 저장 버튼을 클릭

6. 배포하기 

   1. projectName에 들어가서 package.json에 들어간다. 

   2. scripts를 다음과 같이 수정한다. 

      ```json
      "scripts": {
          "start": "react-scripts start",
          "build": "react-scripts build",
          "test": "react-scripts test",
          "eject": "react-scripts eject",
          "deploy": "aws s3 sync ./build s3://user-bucketname --profile=username"
        }
      ```

      deploy 부분의 **<u>user-bucketname</u>**과 **<u>username</u>**을 배포자에 맞게 수정합니다. 

7. CloudFront 설정하기

   1. CloudFront를 설정하는 이유

      - S3의 Custom Domain + HTTPS 지원(기본 S3만 한다면 HTTP만 지원함)
      - CDN(Content Delivery Network)을 통한 더 빠른 페이지 응답속도

   2. 서비스 탭에서 CloudFront를 검색한 뒤 Create Distribution을 클릭합니다. 
      ![AWS CloudFront Create](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS CloudFront Create.png)

      Web 부분의 Get Started 버튼 클릭

   3. Origin Domain Name 부분에서 user-bucketname을 선택합니다. 
      그 후 Default Cache Behavior Settings의 Viewer Protocol Policy에 **<u>Redirect HTTP to HTTPS</u>**를 선택합니다. 
      ![AWS CloudFront Create Distribution](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS CloudFront Create Distribution.png)

   4. Deploy 완료
      ![AWS Deploy Complete](C:\Users\HBY\Desktop\GitHub\document-summary\AWS 관련 정리자료\AWS Deploy Complete.png)

8. 끝으로 배포된 URL을 확인하는 방법은 다음과 같습니다. 

   S3의 정적 웹 사이트 호스팅 속성 탭의 엔드포인트에서 확인이 가능합니다. 

# **이상 읽어주셔서 감사합니다. 잘 배포하세요~ ^^**

