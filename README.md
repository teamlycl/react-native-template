# Lycl React Native Template

## How to

```
npx react-native init AppName --template @teamlycl/react-native-template
```

## Spec

| Library                           | Desc                                                               |
| --------------------------------- | ------------------------------------------------------------------ |
| ruby 2.7.3                        | system ruby 말고, rbenv를 통한 설치가 필요합니다.                  |
| React-native 0.64.1               | -                                                                  |
| React 17.0.1                      | -                                                                  |
| Fastlane                          | ios 인증서 관리 및 앱 테스트 환경 배포 (firebase app distribution) |
|                                   | Fastlane match, gym 사용                                           |
| @react-native-firebase/app 12.0.1 | -                                                                  |

## features
- fastlane 을 이용한 키 동기화 및 빌드
- github action 을 통한 firebase app distribution 배포
- android flavor를 이용한 테스트 환경 구성
- ios prod, dev target을 이용한 환경 구분

## Setting

### git secrets

---

github 설정에서 secrets 값 설정을 우선 완료 해야 합니다.

| secrets id                                                                                                               | value                                                 |
| ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------- |
| **FIREBASE_APP_ID**                                                                                                      | firebase 에서 생성한 android id 값 (dev)                    |
| **FIREBASE_IOS_APP_ID**                                                                                                  | firebase 에서 생성한 ios id 값 (dev)                        |
| **FIREBASE_TOKEN**                                                                                                       | firebase의 token 값                                   |
| fastlane 설치 후 **bundle exec fastlane run firebase_app_distribution_login** 명령어를 통해 로그인 후 token 얻을 수 있음 |
| **APPCENTER_ACCESS_TOKEN**                                                                                               | appcenter token (appcenter 접근을 위해 사용)          |
| **CODEPUSH_DEPLOYMENT_ANDROID_STAGING_KEY**                                                                              | codepush deployment android staging key               |
| **CODEPUSH_DEPLOYMENT_IOS_STAGING_KEY**                                                                                  | codepush deployment ios staging key                   |
| _여기서 부터는 전체 repo에 설정 해두 는 것이 편함_                                                                       |
| **ANDROID_KEY**                                                                                                          | android 배포 키 파일의 base64 값                      |
| **ANDROID_ALIAS**                                                                                                        | android 배포 키의 alias 값                            |
| **ANDROID_PASSWORD**                                                                                                     | android 배포 키의 store, key password                 |
| **GH_TOKEN**                                                                                                             | github token (match 를 통해 key를 가져오는 등 사용됨) |

### Android 설정 및 참고사항

---

1. 안드로이드의 고유의 패키지 명을 설정 해 줍니다 (기본값 com.template)
   1. https://romeoh.tistory.com/entry/React-Native-%ED%8C%A8%ED%82%A4%EC%A7%80%EB%AA%85-%EB%B2%88%EB%93%A4%EB%AA%85-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0-Package-Bundle-Android-iOS
2. Android의 최소 버전은 Android 5.0으로, 빌드 버전은 Android 10 으로 설정되어져 있습니다. 필요시 gradle을 통해 변경 가능합니다.
3. github action 을 통해서 빌드를 하게 되면 자동으로 빌드버전을 상승 시키게 설정 되어져 있습니다.
   1. 만약 1.0(20) 이후 1.1에서 다시 초기화가 필요 할 경우, android/app/src/version.properties 에서 설정 가능 합니다.

### IOS 설정 및 참고사항

1. IOS는 최소 OS를 11로 설정 하였습니다.
2. 작업을 위한 키를 미리 생성해 둔 상태로 진행 해야합니다.
   1. match를 통해 키를 받을수 있단 가정으로 작성 되었습니다
   2. fastlane match adhoc --readonly
3. 안드로이드 처럼 xcode를 이용하여 bundle identifier 를 설정합니다.
   1. 경우에 따라 다양한 bundle identifier를 운영 할 수 있습니다.
4. xcode 에서 아래의 설정을 진행 합니다.
   1. signing -> automatically 해제 후 match 받은 키 값으로 설정 합니다.
   2. targets general 와 project info 에서 iOS 버전을 11.0 으로 맞춥니다.
   3. project build settings 의 versioning를 설정합니다
      1. Current Project Version => 1
      2. Versioning System => Apple Generic
5. 만약 배포 전 키를 변경할 이슈가 있다면, fastlane과 xcode에서 동일하게 맞추어야 합니다.

### codepush 설정 및 참고사항

1. [공식문서](https://docs.microsoft.com/ko-kr/appcenter/distribution/codepush)를 바탕으로 appcenter 설치 후 앱 및 배포트랙을 등록해줍니다.
   1. `npm i -g appcenter-cli`
   2. 로컬에서 appcenter 세팅
      ```
      appcenter login /* 웹 브라우저가 켜지면 앱센터에 로그인해서 키를 복사한 후 커맨드라인에 붙여넣으면 됩니다. */
      appcenter apps create -d {appname} -o {os}(Android/iOS) -p React-Native /* app등록 */
      appcenter codepush deployment add -a {username/appname} {trackname[Staging/Production]} /* 배포트랙등록후 키 저장 */
      ```
   3. appcenter사이트 혹은 cli로 github용 액세스토큰 발급 후 actions secrets안 APPCENTER_ACCESS_TOKEN에 저장해 줍니다.
   4. 배포트랙 등록시 발급된 배포키를 actions secrets안 CODEPUSH*DEPLOYMENT*{ANDROID or IOS}\_{STAGING or PRODUCTION}\_KEY에 저장해 줍니다.
2. [공식문서](https://docs.microsoft.com/ko-kr/appcenter/distribution/codepush/rn-get-started)를 바탕으로 프로젝트 세팅을 진행합니다.
3. 코드푸시 라이브러리를 통해 라벨정보(test시 버전 표기를 위해)를 받아올 수 있습니다.
   ```javascript
   codePush.getUpdateMetadata().then(update => {
     if (update) {
       setLabel(update.label);
     }
   });
   ```

### ##TYPE_PLEASE##

---

IDE에서 전체 검색을 통해 **##TYPE_PLEASE##** 를 입력하여, 나오는 부분에 맞는 값을 입력해 줍니다.

### firebase

---

firebase 에서 ios, android의 서비스를 우선 생성합니다.

상단의 각 OS별 패키지명, 번들 아이디 명을 우선 작업하고 진행해야 합니다.

생성후 각 프로젝트 별로 android/app/google-services.json, ios/template/GoogleService-Info.plist 를 추가해 줍니다.
