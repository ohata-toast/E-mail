## Notification > Email > API v1.4 가이드

[API 도메인]

|환경|	도메인|
|---|---|
|Real|	https://api-mail.cloud.toast.com |

[Header]
```
Content-Type: application/json;charset=UTF-8
```


[curl 예시 주의 사항]

* Windows cmd 에서는 curl 예시가 정상적으로 요청되지 않을 수 있습니다.

## 메일 발송

### 일반 메일 발송

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/sender/mail|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Request body]

|값|	타입|	필수|	설명|
|---|---|---|---|
|​senderAddress|	String|	O|	발신자 메일|
|senderName|	String|	X|	발신자 이름|
|requestDate|	String|	X|	발송 일자 미입력시 : 현재 시간으로 발송 (yyyy-MM-dd HH:mm:ss) |
|title|	String|	O|	제목|
|body|	String|	O|	내용|
|attachFileIdList|	List:Integer|	X|	업로드한 첨부파일 id|
|templateId|	String|	X|	발송 템플릿 ID|
|templateType| String| X| 템플릿 타입 <br/>DEFAULT(default), FREEMARKER)|
|templateParameter|	Object|	X|	치환 파라미터 (메일 제목/내용 치환시 입력)|
|- #key#|	String|	X|	치환 키 (##key##)|
|- #value#|	Object|	X|	치환 키에 매핑되는 Value값|
|receiverList|	List|	O|	수신자 리스트<br/> 최대 1000명까지 발송 가능(받는사람, 참조자 포함)|
|- receiveMailAddr|	String|	O|	수신자 메일주소|
|- receiveName|	String|	X|	수신자 명|
|- receiveType|	String|	O|	수신자 타입 (MRT0 : 받는사람 , MRT1 : 참조, MRT2 : 숨은참조)|
|customHeaders| Map| X| [사용자 지정 헤더](./Overview/#custom-header)|
|userId|	String|	X|	발송 구분자 ex)admin,system|


[주의]

* template을 사용할 경우 title, body는 필수 값이 아닙니다. (입력 시 입력된 값이 template보다 우선 적용)

[예시 1]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/mail -d '{"senderAddress":"support@nhn.com","senderName":"발송자이름","title":"샘플 타이틀","body":"샘플 내용","receiverList":[{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1","receiveType":"MRT0"},{"receiveMailAddr":"customer2@nhn.com","receiveName":"고객2","receiveType":"MRT1"}],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

[예시 2 - 템플릿 사용]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/mail -d '{"templateId":"TEMPLATE1","templateParameter":{"key":"value"},"receiverList":[{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1","receiveType":"MRT0"},{"receiveMailAddr":"customer2@nhn.com","receiveName":"고객2","receiveType":"MRT1"}],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

#### 응답

```
{
    "header": {
        "isSuccessful": boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "data": {
            "requestId": String,
            "results": [
                {
                    "receiveMailAddr": String,
                    "receiveName": String,
                    "receiveType": String,
                    "resultCode": Integer,
                    "resultMessage": String
                }
            ]
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	Object|	데이터 영역|
|-- requestId|	String|	요청 ID|
|-- results|	List|	발송 결과|
|--- receiveMailAddr|	String|	수신자 메일 주소|
|--- receiveName|	String|	수신자 명|
|--- receiveType|	String|	수신자 타입 (MRT0 : 받는사람 , MRT1 : 참조, MRT2 : 숨은참조)|
|--- resultCode|	Integer|	수신자 발송 요청 결과 코드|
|--- resultMessage|	String|	수신자 발송 요청 결과 메시지|

#### v1.4의 달라진 사항

* 미리 템플릿을 등록하지 않아도 **templateType**과 **templateParameter**로 가공한 메일을 제공할 수 있습니다.
* **templateType**에 따라 **templateParameter**이 **title**과 **body**에 적용됩니다.

### 개별 메일 발송

* 다중 수신자에 대해서 수신자 각각에게 메일을 발송하는 기능. 수신자는 수신인이 한명으로 보인다.

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/sender/eachMail||

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Request body]

|값|	타입|	필수|	설명|
|---|---|---|---|
|​senderAddress|	String|	O|	발신자 메일|
|senderName|	String|	X|	발신자 이름|
|requestDate|	String|	X|	발송 일자, 미입력시 : 현재 시간으로 발송 (yyyy-MM-dd HH:mm:ss)|
|title|	String|	O|	제목|
|body|	String|	O|	내용|
|attachFileIdList|	List:Integer|	X|	업로드한 첨부파일 id|
|templateType| String| X| 템플릿 타입 <br/>DEFAULT(default), FREEMARKER)|
|templateId|	String|	X|	발송 템플릿 ID|
|receiverList|	List|	O|	수신자 리스트<br/> 최대 1000명까지 발송 가능|
|- receiveMailAddr|	String|	O|	수신자 메일주소|
|- receiveName|	String|	X|	수신자 명|
|- templateParameter|	Object|	X|	치환 파라미터 (메일 제목/내용 치환시 입력)|
|-- #key#|	String|	X|	치환 키 (##key##)|
|-- #value#|	Object|	X|	치환 키에 매핑되는 Value값|
|customHeaders| Map| X| [사용자 지정 헤더](./Overview/#custom-header)|
|userId|	String|	X|	발송 구분자 ex)admin,system|

[주의]

* template을 사용할 경우 title, body는 필수 값이 아닙니다. (입력 시 입력된 값이 template 보다 우선적용)


[예시 1]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/eachMail -d '{"senderAddress":"support@nhn.com","senderName":"발송자이름","title":"샘플 타이틀","body":"샘플 내용","attachFileIdList":[1, 2],"receiverList":[{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1"}],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

[예시 2 - 템플릿 사용]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/mail -d '{"templateId":"TEMPLATE1","receiverList":[{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1","templateParameter":{"key":"value"}}],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```


#### 응답

```
{
    "header": {
        "isSuccessful": boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "data": {
            "requestId": String,
            "results": [
                {
                    "receiveMailAddr": String,
                    "receiveName": String,
                    "receiveType": String,
                    "resultCode": Integer,
                    "resultMessage": String
                }
            ]
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	Object|	데이터 영역|
|-- requestId|	String|	요청 ID|
|-- results|	List|	발송 결과|
|--- receiveMailAddr|	String|	수신자 메일 주소|
|--- receiveName|	String|	수신자 명|
|--- receiveType|	String|	수신자 타입 (MRT0 : 받는사람 , MRT1 : 참조, MRT2 : 숨은참조) <br>개별 발송은 이 필드를 요청하지 않으므로 null을 반환.|
|--- resultCode|	Integer|	수신자 발송 요청 결과 코드|
|--- resultMessage|	String|	수신자 발송 요청 결과 메시지|


#### v1.4의 달라진 사항

* 미리 템플릿을 등록하지 않아도 **templateType**과 **templateParameter**로 가공한 메일을 제공할 수 있습니다.
* **templateType**에 따라 **templateParameter**이 **title**과 **body**에 적용됩니다.

### 광고성 일반 메일 발송
* URL의 끝에만 ad-mail로 바뀌며 나머지는 일반 메일 발송과 동일하다.

#### 광고메일 전송 시 유의 사항
* 제목에 반드시 (광고) 문구를 삽입하도록 강제하고 있다.
* 자세한 내용은 [[광고성 메일 발송](./console-guide/#_3)]를 참고한다.

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/sender/ad-mail|

[예시 1]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/ad-mail -d '{"senderAddress":"support@nhn.com","senderName":"발송자이름","title":"(광고) 샘플 타이틀","body":"샘플 내용 \n##BLOCK_RECEIVER_LINK## \n##EN_BLOCK_RECEIVER_LINK##","receiverList":[{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1","receiveType":"MRT0"},{"receiveMailAddr":"customer2@nhn.com","receiveName":"고객2","receiveType":"MRT1"}],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

[예시 2 - 템플릿 사용]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/ad-mail -d '{"templateId":"TEMPLATE1","templateParameter":{"key":"value"},"receiverList":[{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1","receiveType":"MRT0"},{"receiveMailAddr":"customer2@nhn.com","receiveName":"고객2","receiveType":"MRT1"}],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

### 광고성 개별 메일 발송

* URL의 끝에만 ad-eachMail로 바뀌며 나머지는 개별 메일 발송과 동일하다.

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/sender/ad-eachMail |

[예시 1]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/ad-eachMail -d '{"senderAddress":"support@nhn.com","senderName":"발송자이름","title":"(광고) 샘플 타이틀","body":"샘플 내용 \n##BLOCK_RECEIVER_LINK## \n##EN_BLOCK_RECEIVER_LINK##","attachFileIdList":[1, 2],"receiverList":[{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1"}],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

[예시 2 - 템플릿 사용]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/ad-eachMail -d '{"templateId":"TEMPLATE1","receiverList":[{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1","templateParameter":{"key":"value"}}],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

### 인증 메일 발송

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/sender/auth-mail||

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Request body]

|값|	타입|	필수|	설명|
|---|---|---|---|
|​senderAddress|	String|	O|	발신자 메일|
|senderName|	String|	X|	발신자 이름|
|requestDate|	String|	X|	발송 일자, 미입력시 : 현재 시간으로 발송 (yyyy-MM-dd HH:mm:ss)|
|title|	String|	O|	제목|
|body|	String|	O|	내용|
|templateId|	String|	X|	발송 템플릿 ID|
|receiver|	Object|	O|	수신자 |
|- receiveMailAddr|	String|	O|	수신자 메일주소|
|- receiveName|	String|	X|	수신자 명|
|- templateParameter|	Object|	X|	치환 파라미터 (메일 제목/내용 치환시 입력)|
|-- #key#|	String|	X|	치환 키 (##key##)|
|-- #value#|	Object|	X|	치환 키에 매핑되는 Value값|
|customHeaders| Map| X| [사용자 지정 헤더](./Overview/#custom-header)|
|userId|	String|	X|	발송 구분자 ex)admin,system|

[주의]

* template을 사용할 경우 title, body는 필수 값이 아닙니다. (입력 시 입력된 값이 template 보다 우선적용)

#### 일반 메일과 다른 점
인증 메일 성격상 다음과 같이 다른 특성들이 있습니다.

* 단건 발송(1명의 수신자)만 가능합니다.
* 첨부 파일 기능을 지원하지 않습니다. 첨부 파일이 포함된 템플릿은 지원하지 않습니다.

[예시 1]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/auth-mail -d '{"senderAddress":"support@nhn.com","senderName":"발송자이름","title":"샘플 타이틀","body":"샘플 내용","receiver":{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1"},"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

[예시 2 - 템플릿 사용]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/auth-mail -d '{"templateId":"TEMPLATE1","receiver":{"receiveMailAddr":"customer1@nhn.com","receiveName":"고객1","templateParameter":{"key":"value"}},"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

#### 응답

```
{
    "header": {
        "isSuccessful":boolean,
        "resultCode":Integer,
        "resultMessage":String
    },
    "body": {
        "data": {
            "requestId":String,
            "statusCode":String
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	Object|	데이터 영역|
|-- requestId|	String|	요청 ID|
|-- statusCode|	String|	요청 상태 코드 (Y: 발송준비 , N : 발송준비실패)|

### 태그 메일 발송

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/sender/tagMail|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Request body]

```
{
    "senderAddress": String,
    "senderName": String,
    "requestDate": String,
    "title": String,
    "body": String,
    "templateId": String,
    "adYn": String,
    "autoSendYn": String,
    "attachFileIdList": List:String,
    "tagExpression": List:String,
    "userId": String
}
```

|값|	타입|	필수|	설명|
| - | - | - | - |
| senderAddress  | String | O|발신자 주소 |
| senderName | String | X|발신자명 |
| requestDate | String | X|발송 일자 미입력시 : 현재 시간으로 발송 (yyyy-MM-dd HH:mm:ss) |
| title  | String | O|제목 |
| body  | String | O|내용 |
| templateId  | String | X|템플릿 ID |
| adYn  | String | X|광고 여부 (default 'N') |
| autoSendYn  | String | X|자동 발송 여부 (default 'Y') |
| attachFileIdList  | List:Integer | X|첨부파일 리스트 |
| tagExpression  | List:String | O|태그 표현식 |
|customHeaders| Map| X| [사용자 지정 헤더](./Overview/#custom-header)|
| userId  | String | X|발송 구분자 ex)admin,system|

[주의]

* template을 사용할 경우 title, body는 필수 값이 아닙니다. (입력 시 입력된 값이 template보다 우선 적용)


[예시 1]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/tagMail -d '{"senderAddress":"support@nhn.com","senderName":"발송자이름","title":"샘플 타이틀","body":"샘플 내용","attachFileIdList":[1, 2],"tagExpression":["tag1","AND","tag2"],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

[예시 2 - 템플릿 사용]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/tagMail -d '{"templateId":"TEMPLATE1","tagExpression":["tag1","AND","tag2"],"customHeaders":{"X-Sample":"sample","Content-Type":"text/html; charset=utf-8"},"userId":"XXXXX"}'
```

#### 응답

```
{
    "header": {
        "isSuccessful":boolean,
        "resultCode":Integer,
        "resultMessage":String
    },
    "body": {
        "data": {
            "requestId":String
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	Object|	데이터 영역|
|-- requestId|	String|	요청 ID|

### 첨부파일 업로드

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/attachfile/binaryUpload|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Request body]

```
{
    "fileName": String,
    "createUser": String,
    "fileBody": Byte[]
}
```

|값|	타입|	필수|	설명|
|---|---|---|---|
|fileName|	String|	O|	파일이름|
|fileBody|	Byte[]|	O|	파일의 Byte[] 값|
|createUser|	String|	O|	파일 업로드 유저 정보|

[예시]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/attachfile/binaryUpload -d '{"fileName":"file.csv","createUser":"XXXXX","fileBody":[]}'
```

#### 응답

```
{
  "header": {
    "isSuccessful":  Boolean,
    "resultCode": Integer,
    "resultMessage": String
  },
  "body": {
    "data": {
      "fileId": Integer,
      "fileName": String
    }
  }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	Object|	데이터 영역|
|-- fileId|	String|	파일 ID|
|-- fileName|	String|	파일명|


### 제목/본문 치환

* v1.4부터 발송 API에 추가된 **templateType**따라 다른 방식으로 제목과 본문이 가공됩니다.
#### 기본 타입
* (##치환 Key##) 형식으로 입력하면 사용자가 입력한 **templateParameter**로 치환할 수 있습니다.
```
* title : ##title_name##님 안녕하세요 !!
* body : ##body_content## 발송합니다.
->
* title : 클라우드고객1님 안녕하세요!!
* body : test2 발송합니다.
```

#### FreeMarker 타입
* [FreeMarker 템플릿 엔진](https://freemarker.apache.org/)을 지원합니다.
* 템플릿 언어를 사용하여 사용자가 입력한 **templateParameter**로 치환할 수 있습니다.
```
* title : ${title_name}님 안녕하세요 !!
* body : ${body_content} 발송합니다.
->
* title : 클라우드고객1님 안녕하세요!!
* body : test2 발송합니다.
```

#### 일반 메일요청 예시
```
{
    "senderAddress" : "support@nhn.com",
    "templateId": "template1",
    "templateParameter" : {"title_name": "클라우드고객1", "body_content": "test1"},
    "receiverList" : [
        {
            "receiveMailAddr" : "customer1@nhn.com",
            "receiveType" : "MRT0"
        }

    ],
    "userId" : "tester"
}
```

#### 개별 메일요청 예시
```
{
    "senderAddress" : "support@nhn.com",
    "templateId": "template1",
    "receiverList" : [
        {
            "receiveMailAddr" : "customer1@nhn.com",
            "templateParameter" : {"title_name": "클라우드고객1", "body_content": "test1"}
        },
        {
            "receiveMailAddr" : "customer2@nhn.com",
            "templateParameter" : {"title_name": "클라우드고객2", "body_content": "test2"}
        }

    ],
    "userId" : "tester"
}
```

## 메일 조회

### 메일 발송 리스트 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/sender/mails|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Query parameter]

|값|	타입|	필수|	설명|
|---|---|---|---|
|requestId|	String|	O|	요청 ID|
|startSendDate|	String|	O|	발송 날짜 시작 값(yyyy-MM-dd HH:mm:ss)|
|endSendDate|	String|	X|	발송 날짜 종료 값(yyyy-MM-dd HH:mm:ss)|
|startReceiveDate|	String|	X|	수신 날짜 시작 값(yyyy-MM-dd HH:mm:ss)|
|endReceiveDate|	String|	X|	수신 날짜 종료 값(yyyy-MM-dd HH:mm:ss)|
|senderMail|	String|	X|	발신메일 주소|
|senderName|	String|	X|	발신자 이름|
|receiveMail|	String|	X|	수신메일 주소|
|templateId|	String|	X|	템플릿번호|
|sendStatus|	String|	X|	발송상태 코드 <br/> SST0:발송준비, SST1:발송중,  <br/> SST2:발송완료, SST3 : 발송실패|
|pageNum|	Integer|	X|	페이지 번호(Default : 1)|
|pageSize|	Integer|	X|	조회 건수(Default : 15)|

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/mails?startSendDate=2018-03-01+00%3A00&endSendDate=2018-03-07+23%3A59&pageSize=10"
```

#### 응답

```
{
    "header": {
        "isSuccessful": boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "pageNum": Integer,
        "pageSize": Integer,
        "totalCount": Integer,
        "data": [{
            "requestId": String,
            "requestIp": String,
            "templateId": String,
            "templateName": String,
            "masterStatusCode": String,
            "mailSeq": String,
            "senderName": String,
            "senderMail": String,
            "title": String,
            "body": String,
            "resultId": String,
            "resultDate": String,
            "mailStatusCode": String,
            "mailStatusName": String,
            "requestDate": String,
            "receiverMail": String,
            "receiveType": String
        }]
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- pageNum|	Integer|	현재 페이지 번호|
|-pageSize|	Integer|	조회된 데이터 건수|
|- totalCount|	Integer|	총 데이터 건수|
|- data|	List|	데이터 영역|
|-- requestId|	String|	요청 ID|
|-- requestDate|	String|	발신일시|
|-- requestIp|	String|	발신서버 IP|
|-- resultDate|	String|	수신일시|
|-- templateId|	String|	템플릿ID|
|-- templateName|	String|	템플릿명|
|-- masterStatusCode|	String|	메일 발송 준비 상태코드 ( "Y" : 발송준비 , "N" : 발송실패)|
|-- mailSeq|	String|	메일 순번|
|-- body|	String|	본문내용|
|-- title|	String|	메일 제목|
|-- senderMail|	String|	발신자 메일주소|
|-- senderName|	String|	발신자 이름|
|-- receiverMail|	String|	수신자 메일주소|
|-- receiveType|	String|	수신자 타입 (MRT0 : 받는사람 , MRT1 : 참조, MRT2 : 숨은참조)|
|-- resultId|	String|	발송결과 ID|
|-- resultDate|	String|	발송 완료 일시|
|-- mailStatusCode|	String|	발송상태 코드 <br/> SST0:발송준비, SST1:발송중,  <br/> SST2:발송완료, SST3 : 발송실패|
|-- mailStatusName|	String|	발송상태명|
|-- requestDate|	String|	발송 요청일시|

### 메일 발송 상세 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/sender/mail/{requestId}/{mailSeq}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|requestId|	String|	요청ID|
|mailSeq|	Integer|	메일 순번|

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/sender/mail/{requestId}/{mailSeq}"
```

#### 응답

```
{
    "header": {
        "isSuccessful": boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "data": {
            "requestId": String,
            "requestIp": String,
            "requestDate": String,
            "masterStatusCode": String,
            "mailStatusCode": String,
            "mailStatusName": String,
            "templateId": String,
            "templateName": String,
            "senderName": String,
            "senderMail": String,
            "title": String,
            "body": String,
            "attachFileYn": String,
            "resultId": String,
            "resultDate": String,
            "receivers": [{
                "requestId": String,
                "mailSequence": Integer,
                "receiveType": String,
                "receiveTypeName": String,
                "receiveName": String,
                "receiveMailAddr": String,
                "readYn": String,
                "readDate": String
            }],
            "attachFileList": [{
                "fileType": String,
                "fileId": Integer,
                "fileName": String,
                "filePath": String,
                "fileSize": Integer,
                "createDate": String
            }],
            "customHeaders": Map
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	Object|	데이터 영역|
|-- mailStatusCode|	String|	발송상태|
|-- mailStatusName|	String|	발송상태명|
|-- templateId|	String|	템플릿ID|
|-- templateName|	String|	템플릿 명|
|-- senderName|	String|	발신자 이름|
|-- senderMail|	String|	발신 메일주소|
|-- title|	String|	메일 제목|
|-- body|	String|	메일 내용|
|-- attachFileYn|	String|	첨부파일 여부|
|-- resultId|	String|	메일 발송 ID|
|-- resultDate|	String|	발송완료일시|
|-- receivers|	List|	수신자 리스트|
|--- requestId|	String|	요청ID|
|--- mailSequence|	Integer|	메일 순번|
|--- receiveType|	String|	수신자 타입|
|--- receiveTypeName|	String|	수신자 타입명|
|--- receiveName|	String|	수신자 이름|
|--- receiveMailAddr|	String|	수신 메일 주소|
|--- readYn| String| 수신 여부 |
|--- readDate| String| 수신 날짜(yyyy-MM-dd HH:mm:ss.SSS)|
|-- attachFileList|	List|	첨부파일 리스트|
|--- fileType|	String|	첨부파일 타입 (MAIL: 메일에 첨부된 파일, TEMPLATE: 템플릿에 첨부된 파일)|
|--- fileId| String| 파일ID|
|--- fileName|	String|	첨부파일 이름|
|--- filePath|	String|	첨부파일 경로|
|--- fileSize|	Integer|	첨부파일 크기 (byte)|
|--- createDate|	String|	생성일시|
|-- customHeaders|	Map|	[사용자 지정 헤더](./Overview/#custom-header) |


### 태그 메일 발송 요청 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/tagMails|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Query parameter]

|값|	타입|	필수|	설명|
|---|---|---|---|
|requestId|	String|	O|	요청 ID|
|startSendDate|	String|	O|	발송 날짜 시작 값(yyyy-MM-dd HH:mm:ss)|
|endSendDate|	String|	O|	발송 날짜 종료 값(yyyy-MM-dd HH:mm:ss)|
|senderMail|	String|	X|	발신메일 주소|
|senderName|	String|	X|	발신자 이름|
|templateId|	String|	X|	템플릿 ID|
|sendStatus|	String|	X|	발송상태 코드 <br/> WAIT: 대기, READY: 발송준비, <br/>SENDREADY: 발송준비완료, SENDWAIT: 발송대기, <br/>SENDING: 발송중, COMPLETE: 발송완료, <br/>FAIL: 발송실패, CANCEL: 발송취소|
|pageNum|	Integer|	X|	페이지 번호(Default : 1)|
|pageSize|	Integer|	X|	조회 건수(Default : 15)|

[주의]

* requestId가 있는 경우, startSendDate와 endSendDate는 필수 값이 아닙니다.
* startSendDate와 endSendDate가 있는 경우, requestId는 필수 값이 아닙니다.

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/tagMails?startSendDate=2018-03-01+00%3A00&endSendDate=2018-03-07+23%3A59&pageSize=10"
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "pageNum": Integer,
        "pageSize": Integer,
        "totalCount": Integer,
        "data": [
            {
                "requestId": String,
                "requestIp": String,
                "requestDate": String,
                "tagSendStatus": String,
                "tagExpression": List:String,
                "templateId": String,
                "templateName": String,
                "senderName": String,
                "senderMailAddress": String,
                "title": String,
                "body": String,
                "attachYn": String,
                "adYn": String,
                "createUser": String,
                "createDate": String,
                "updateUser": String,
                "updateDate": String
            }
        ]
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- pageNum|	Integer|	현재 페이지 번호|
|-pageSize|	Integer|	조회된 데이터 건수|
|- totalCount|	Integer|	총 데이터 건수|
|- data|	List|	데이터 영역|
|-- requestId | String  | 요청 ID |
|-- requestIp |  String  | 요청 아이피 |
|-- requestDate |  String  | 요청 시간 |
|-- tagSendStatus |  String  | 발송상태 코드 <br/> WAIT: 대기, READY: 발송준비, <br/>SENDREADY: 발송준비완료, SENDWAIT: 발송대기, <br/>SENDING: 발송중, COMPLETE: 발송완료, <br/>FAIL: 발송실패, CANCEL: 발송취소 |
|-- tagExpression |  List:String  | 태그 표현식 |
|-- templateId |  String  | 템플릿 ID |
|-- templateName |  String  | 템플릿명 |
|-- senderName |  String  | 발신자명 |
|-- senderMail |  String  | 발신자주소 |
|-- title |  String  | 제목 |
|-- body |  String  | 내용 |
|-- attachYn |  String  | 첨부파일여부 |
|-- adYn |  String  | 광고여부 |
|-- createUser |  String  | 생성자 |
|-- createDate |  String  | 생성일시 |
|-- updateUser |  String  | 수정자 |
|-- updateDate |  String  | 수정일시 |

### 태그 메일 발송 수신자 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/tagMails/{requestId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|requestId|	String|	요청 ID|

[Query parameter]

|값|	타입|	필수|	설명|
|---|---|---|---|
|receiveMail|	String|	X|	수신 메일 주소|
|startReceiveDate|	String|	X|	수신 날짜 시작 값(yyyy-MM-dd HH:mm:ss)|
|endReceiveDate|	String|	X|	수신 날짜 종료 값(yyyy-MM-dd HH:mm:ss)|
|receiveStatus|	String|	X|	발송상태 코드 <br/> SST0:발송준비, SST1:발송중,  <br/> SST2:발송완료, SST3 : 발송실패|
|pageNum|	Integer|	X|	페이지 번호(Default : 1)|
|pageSize|	Integer|	X|	조회 건수(Default : 15)|

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/tagMails/{requestId}?startReceiveDate=2018-03-01+00%3A00&endReceiveDate=2018-03-07+23%3A59&pageSize=10"
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "pageNum": Integer,
        "pageSize": Integer,
        "totalCount": Integer,
        "data": [
            {
                "requestId": String,
                "mailSequence": Integer,
                "receiveMail": String,
                "mailStatusCode": String,
                "mailStatusName": String,
                "resultId": String,
                "resultDate": String,
                "readYn": String,
                "readDate": String,
                "createUser": String,
                "createDate": String,
                "updateUser": String,
                "updateDate": String
            }
        ]
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- pageNum|	Integer|	현재 페이지 번호|
|-pageSize|	Integer|	조회된 데이터 건수|
|- totalCount|	Integer|	총 데이터 건수|
|- data|	List|	데이터 영역|
|-- requestId | String  | 요청 ID |
|-- mailSequence | Integer  | 메일 순번 |
|-- receiveMail | String  | 수신자주소 |
|-- mailStatusCode | String  | 메일 상태 코드 <br/> SST0:발송준비, SST1:발송중,  <br/> SST2:발송완료, SST3 : 발송실패|
|-- mailStatusName | String  | 메일 상태명 |
|-- resultId | String  | SMTP ID |
|-- resultDate | String  | 실제 발송 시간 |
|-- readYn | String  | 읽음 여부 |
|-- readDate | String  | 읽은 시간 |
|-- createUser |  String  | 생성자 |
|-- createDate |  String  | 생성일시 |
|-- updateUser |  String  | 수정자 |
|-- updateDate |  String  | 수정일시 |

### 태그 메일 발송 상세 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/tagMails/{requestId}/{mailSequence}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|requestId|	String|	요청 ID|
|mailSequence|	Integer|	메일 순번|

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/tagMails/{requestId}/{mailSequence}"
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "data": {
            "requestId": String,
            "requestIp": String,
            "templateId": String,
            "templateName": String,
            "mailStatusCode": String,
            "mailStatusName": String,
            "requestDate": String,
            "resultDate": String,
            "senderName": String,
            "senderMail": String",
            "resultId": String,
            "title": String,
            "body": String",
            "receivers": [
                {
                    "requestId": String
                    "mailSequence": String
                    "receiveType": String
                    "receiveTypeName": String
                    "receiveMailAddr": String
                    "readYn": String
                    "readDate": String
                }
            ],
            "attachFileList": [
                {
                    "fileType": String,
                    "fileId": Integer
                    "fileName": String,
                    "filePath": String,
                    "fileSize": Integer,
                    "createDate": String
                }
            ],
            "customHeaders": Map
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- pageNum|	Integer|	현재 페이지 번호|
|-pageSize|	Integer|	조회된 데이터 건수|
|- totalCount|	Integer|	총 데이터 건수|
|- data|	List|	데이터 영역|
|-- requestId  | String  | 요청 ID |
|-- requestIp | String  | 요청 IP |
|-- templateId | String  | 템플릿 ID |
|-- templateName | String  | 템플릿 명 |
|-- mailStatusCode | String  | 메일 상태 코드 <br/> SST0:발송준비, SST1:발송중,  <br/> SST2:발송완료, SST3 : 발송실패 |
|-- mailStatusName | String  | 메일 상태 명 |
|-- requestDate | String  | 요청 시간 |
|-- resultDate | String  | 결과 시간 |
|-- senderName | String  | 발신자 명 |
|-- senderMail | String  | 발신자 주소 |
|-- resultId | String  | SMTP ID |
|-- title | String  | 제목 |
|-- body | String  | 내용 |
|-- receivers | List| 수신자 리스트|
|--- requestId | String  | 요청 ID |
|--- mailSequence | Integer  | 메일 순번 |
|--- receiveType | String  | 수신자 타입 (MRT0 : 받는사람 , MRT1 : 참조, MRT2 : 숨은참조) |
|--- receiveTypeName | String  | 수신자 타입명 |
|--- receiveMailAddr | String  | 수신자 메일 주소 |
|--- readYn | String  | 읽음 여부 |
|--- readDate | String  | 읽은 시간 |
|-- attachFileList | List  | 첨부파일 리스트 |
|--- fileType|	String|	첨부파일 타입 (MAIL: 메일에 첨부된 파일, TEMPLATE: 템플릿에 첨부된 파일)|
|--- fileId| String| 파일ID|
|--- fileName|	String|	첨부파일 이름|
|--- filePath|	String|	첨부파일 경로|
|--- fileSize|	Integer|	첨부파일 크기 (byte)|
|--- createDate|	String|	생성일시|
|-- customHeaders|	Map|	[사용자 지정 헤더](./Overview/#custom-header) |

<p id="category"></p>

## 카테고리 관리

### 카테고리 리스트 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	GET /email/v1.4/appKeys/{appKey}/categories|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Query parameter]

|값|	타입|	필수|	설명|
|---|---|---|---|
|useYn|	String|	X|	사용 여부 Y, N |
|categoryParentId|	Integer|	X|	부모 카테고리 ID |
|pageNum|	Integer|	X|	페이지 번호(Default : 1)|
|pageSize|	Integer|	X|	조회 건수(Default : 15)|

[예시]
``` sh
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/categories?useYn=Y&categoryParentId=1&pageNum=1&pageSize=10"
```

#### 응답

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    },
    "body": {
        "pageNum": 1,
        "pageSize": 15,
        "totalCount": 3,
        "data": [
            {
                "categoryId": 12345,
                "categoryParentId": 0,
                "depth": 0,
                "categoryName": "Category",
                "categoryDesc": "Top Category",
                "useYn": "Y",
                "createUser": "user",
                "createDate": "2019-07-23 00:00:00.0",
                "updateUser": "user",
                "updateDate": "2019-07-23 00:00:00.0",
            }
        ]
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- pageNum|	Integer|	현재 페이지 번호|
|- pageSize|	Integer|	조회된 데이터 건수|
|- totalCount|	Integer|	총 데이터 건수|
|- data|	List|	데이터 영역|
|-- categoryId|	Integer|	카테고리 ID|
|-- categoryParentId|	Integer| 부모 카테고리 ID (최상위 카테고리인 경우 0)|
|-- depth|	Integer| depth (최상위 카테고리인 경우 0) |
|-- categoryName|	String|	카테고리 명|
|-- categoryDesc|	String|	카테고리 설명|
|-- useYn|	String|	사용여부|
|-- createUser|	String|	생성자|
|-- createDate|	String|	생성일시|
|-- updateUser|	String|	수정자|
|-- updateDate|	String|	수정일시|

### 카테고리 상세 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/categories/{categoryId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|categoryId|	String|	카테고리 ID|

[예시]
``` sh
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/categories/{categoryId}"
```

#### 응답

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    },
    "body": {
        "data": {
            "categoryId": 12345,
            "categoryParentId": 0,
            "depth": 0,
            "categoryName": "Category",
            "categoryDesc": "Top Category",
            "useYn": "Y",
            "createUser": "user",
            "createDate": "2019-07-23 00:00:00.0",
            "updateUser": "user",
            "updateDate": "2019-07-23 00:00:00.0"
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	List|	데이터 영역|
|-- categoryId|	Integer|	카테고리 ID|
|-- categoryParentId|	Integer| 부모 카테고리 ID (최상위 카테고리인 경우 0)|
|-- depth|	Integer| depth (최상위 카테고리인 경우 0) |
|-- categoryName|	String|	카테고리 명|
|-- categoryDesc|	String|	카테고리 설명|
|-- useYn|	String|	사용여부|
|-- createUser|	String|	생성자|
|-- createDate|	String|	생성일시|
|-- updateUser|	String|	수정자|
|-- updateDate|	String|	수정일시|


### 카테고리 등록

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|/email/v1.4/appKeys/{appKey}/categories|


[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 앱키|

[Request body]

|값|	타입|	최대 길이 | 필수|	설명|
|---|---|---|---|---|
| categoryParentId |	Integer|	- | X |	부모 카테고리 ID <br/> 최상위 카테고리 ID(default) |
| categoryName |	String|	200 | O |	카테고리 이름 |
| categoryDesc |	String| 1000 |	X |	카테고리 설명|
| useYn |	String| 1 |	X|	사용 여부 Y(default), N|
| userId | String | 50 | X | 사용자 ID |

[예시]
``` sh
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/categories -d '{"categoryParentId":12345,"categoryName":"Category","categoryDesc":"Top Category","useYn":"Y","userId":"USER"}'
```


#### 응답

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    },
    "body": {
        "data": {
            "categoryId": 12346
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	Object|	데이터 영역|
|-- categoryId|	Integer|	카테고리 ID|


### 카테고리 수정

#### 요청

[URL]

|Http method|	URI|
|---|---|
|PUT|/email/v1.4/appKeys/{appKey}/categories/{categoryId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 앱키|
|categoryId|	Integer|	카테고리 ID|

[Request body]

|값|	타입|	최대 길이 | 필수|	설명|
|---|---|---|---|---|
| categoryName |	String|	200 | X |	카테고리 이름 |
| categoryDesc |	String| 1000 |	X |	카테고리 설명|
| useYn |	String| 1 |	X|	사용 여부 Y, N|
| userId | String | 50 | X | 사용자 ID |

[예시]
``` sh
curl -X PUT -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/categories/{categoryId} -d '{"categoryName":"Category","categoryDesc":"Top Category","useYn":"Y","userId":"USER"}'
```

#### 응답

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

### 카테고리 삭제

#### 요청

[URL]

|Http method|	URI|
|---|---|
|DELETE|/email/v1.4/appKeys/{appKey}/categories/{categoryId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 앱키|
|categoryId|	Integer|	카테고리 ID|

[예시]
``` sh
curl -X DELETE -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/categories/{categoryId}
```

#### 응답

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

<p id="template"></p>

## 템플릿 관리

### 템플릿 리스트 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/templates|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Query parameter]

|값|	타입|	필수|	설명|
|---|---|---|---|
|categoryId|	Integer|	X|	카테고리 ID|
|useYn|	String|	X|	사용 여부(Y/N)|
|pageNum|	Integer|	X|	페이지 번호(Default : 1)|
|pageSize|	Integer|	X|	조회 건수(Default : 15)|

[예시]
``` sh
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/templates?useYn=Y&pageNum=1&pageSize=10"
```

#### 응답

```
{
    "header": {
        "resultCode": Integer,
        "resultMessage": String,
        "isSuccessful": Boolean
    },
    "body": {
        "pageNum": Integer,
        "pageSize": Integer,
        "totalCount": Integer,
        "data": [
            {
                "templateId": String,
                "categoryId": Integer,
                "categoryName": String,
                "templateName": String,
                "templateDesc": String,
                "useYn": String,
                "delYn": String,
                "title": String,
                "createDate": String,
                "updateDate": String
            }
        ]
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- pageNum|	Integer|	현재 페이지 번호|
|- pageSize|	Integer|	조회된 데이터 건수|
|- totalCount|	Integer|	총 데이터 건수|
|- data|	List|	데이터 영역|
|-- templateId|	String|	템플릿 ID|
|-- categoryId|	Integer|	카테고리 ID|
|-- categoryName|	String|	카테고리 명|
|-- templateName|	String|	템플릿 명|
|-- templateDesc|	String|	템플릿 설명|
|-- useYn|	String|	사용여부|
|-- delYn|	String|	삭제 여부|
|-- title|	String|	메일 제목|
|-- createDate|	String|	생성일시|
|-- updateDate|	String|	수정일시|

### 템플릿 상세 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/templates/{templateId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|templateId|	String|	템플릿 ID|

[예시]
``` sh
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/templates/{templateId}"
```

#### 응답

```
{
    "header": {
        "resultCode": Integer,
        "resultMessage": String,
        "isSuccessful": Boolean
    },
    "body": {
        "data": {
            "templateId": String,
            "categoryId": Integer,
            "categoryName": String,
            "templateName": String,
            "templateDesc": String,
            "useYn": String,
            "delYn": String,
            "sendMailAddress": String,
            "title": String,
            "templateType": String,
            "body": String,
            "createDate": String,
            "updateDate": String,
            "attachFileList": [
                {
                    "fileType": String,
                    "fileId": Integer,
                    "fileName": String,
                    "filePath": String,
                    "fileSize": Integer,
                    "createDate": String
                }
            ]
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	List|	데이터 영역|
|-- templateId|	String|	템플릿 ID|
|-- categoryId|	Integer|	카테고리 ID|
|-- categoryName|	String|	카테고리명|
|-- templateName|	String|	템플릿명|
|-- templateDesc|	String|	템플릿 설명|
|-- useYn|	String|	사용여부 (Y= 사용중, N= 사용안함)|
|-- delYn|	String|	삭제 여부(Y= 삭제, N= 삭제 아님)|
|-- sendMailAddress|	String|	발신메일주소|
|-- title|	String|	메일 주소|
|-- templateType|	String|	템플릿 타입 <br/>DEFAULT(default), FREEMARKER)|
|-- body|	String|	메일 내용|
|-- createDate|	String|	생성일시|
|-- updateDate|	String|	수정일시|
|-- attachFileList|	List|	첨부파일 리스트|
|--- fileType|	String|	첨부파일 타입 (MAIL: 메일에 첨부된 파일, TEMPLATE: 템플릿에 첨부된 파일)|
|--- fileId| String| 파일 ID|
|--- fileName|	String|	첨부파일 이름|
|--- filePath|	String|	첨부파일 경로|
|--- fileSize|	Integer|	첨부파일 크기 (byte)|
|--- createDate|	String|	생성일시|


### 템플릿 등록

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/templates|


[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 앱키|

[Request body]

|값|	타입|	최대 길이 | 필수|	설명|
|---|---|---|---|---|
| categoryId |	Integer|	- | O |	카테고리 ID |
| templateId | String | 50 | O | 템플릿 ID |
| templateName |	String| 200 |	O |	템플릿명|
| templateDesc |	String| 4000 |	X |	템플릿 설명|
| useYn |	String| 1 |	X|	사용 여부 Y(default), N|
| sendMailAddress | String| 300 | O| 발신 메일 주소 |
| title | String | 500 | O | 메일 제목 |
| templateType |	String| 10 |	X|	템플릿 타입 <br/>DEFAULT(default), FREEMARKER)|
| body | String | - | O | 메일 본문 |
| attachFileIdList | List<Integer> | - | X | 첨부 파일 ID(fileId) |
| userId | String | 50 | X | 사용자 ID |

[예시]
``` sh
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/templates -d '{"categoryId":1,"templateId":"TEAMPLTE_ID","templateName":"템플릿 이름","templateDesc":"템플릿 설명","useYn":"Y","sendMailAddress":"test@nhn.com","title":"메일 제목","templateType":"DEFAULT","body":"메일 내용","attachFileIdList":[1,2,3],"userId":"USER"}'
```


#### 응답

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

### 템플릿 첨부 파일 업로드

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/templates/attachfile/binaryUpload|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Request body]

|값|	타입|	최대 길이| 필수|	설명|
|---|---|---|---|---|
|fileName|	String|	100|O|	파일이름|
|fileBody|	Byte[]|	- |O|	파일의 Byte[] 값|
|userId|	String|	50|X|	유저 ID|

[예시]
``` sh
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/templates/attachfile/binaryUpload -d '{"fileName":"file.csv","userId":"USER","fileBody":[]}'
```

#### 응답

``` json
{
  "header": {
    "isSuccessful":  true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {
    "data": {
      "fileId": 1,
      "fileName": "file.csv"
    }
  }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	Object|	데이터 영역|
|-- fileId|	String|	파일 ID|
|-- fileName|	String|	파일명|

[주의]

* 파일을 업로드 한 다음 템플릿에 첨부하면, 다른 템플릿에 그 파일을 첨부할 수 없습니다. 첨부된 파일을 수정하거나 새로 업로드 한 후 첨부해야 합니다.

### 템플릿 수정

#### 요청

[URL]

|Http method|	URI|
|---|---|
|PUT|	/email/v1.4/appKeys/{appKey}/templates/{templateId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 앱키|
|templateId|	String|	템플릿 ID|

[Request body]

|값|	타입|	최대 길이 | 필수|	설명|
|---|---|---|---|---|
| templateName |	String| 200 |	X |	템플릿명|
| templateDesc |	String| 4000 |	X |	템플릿 설명|
| useYn |	String| 1 |	X |	사용 여부 Y, N|
| sendMailAddress | String| 300 | X| 발신 메일 주소 |
| title | String | 500 | X | 메일 제목 |
| templateType |	String| 10 |	X|	템플릿 타입 <br/>DEFAULT, FREEMARKER)|
| body | String | - | X | 메일 본문 |
| attachFileIdList | List<Integer> | - | X | 첨부 파일 ID(fileId) |
| userId | String | 50 | X | 사용자 ID |

[예시]
``` sh
curl -X PUT -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/templates/{templateId} -d '{"templateName":"템플릿 이름","templateDesc":"템플릿 설명","useYn":"Y","sendMailAddress":"test@nhn.com","title":"메일 제목","templateType":"DEFAULT","body":"메일 내용","attachFileIdList":[1,2,3],"userId":"USER"}'
```

#### 응답

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

### 템플릿 삭제

#### 요청

[URL]

|Http method|	URI|
|---|---|
|DELETE|	/email/v1.4/appKeys/{appKey}/templates/{templateId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 앱키|
|templateId|	String|	템플릿 ID|

[예시]
``` sh
curl -X DELETE -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/templates/{templateId}
```

#### 응답

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|


## 태그 관리

### 태그 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/tags|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Query parameter]

|값|	타입|	필수|	설명|
|---|---|---|---|
|pageNum|	Integer|	X|	페이지 번호(Default : 1)|
|pageSize|	Integer|	X|	조회 건수(Default : 15)|

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/tags?pageNum=1&pageSize=10"
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "pageNum": Integer,
        "pageSize": Integer,
        "totalCount": Integer,
        "data": [
            {
                "tagId": String,
                "tagName": String,
                "createdDate": String,
                "updatedDate": String
            }
        ]
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	List|	데이터 영역|
|-- tagId | String | 태그 ID |
|-- tagName | String | 태그 이름 |
|-- createdDate | String | 생성일시 |
|-- updatedDate | String | 수정일시 |

### 태그 등록

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/tags|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Request body]

|값|	타입|	필수|	설명|
|---|---|---|---|
|tagName|	String|	O|	태그 이름|

[예시]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/tags -d '{"tagName":"샘플태그"}'
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "data": {
            "tagId": String
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	List|	데이터 영역|
|-- tagId | String | 태그 ID |

### 태그 수정

#### 요청

[URL]

|Http method|	URI|
|---|---|
|PUT|	/email/v1.4/appKeys/{appKey}/tags/{tagId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|tagId|	String|	태그 ID|

[Request body]

|값|	타입|	필수|	설명|
|---|---|---|---|
|tagName|	String|	O|	태그 이름|

[예시]
```
curl -X PUT -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/tags/{tagId} -d '{"tagName":"샘플태그2"}'
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

### 태그 삭제

#### 요청

[URL]

|Http method|	URI|
|---|---|
|DELETE|	/email/v1.4/appKeys/{appKey}/tags/{tagId}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|tagId|	String|	태그 ID|

[예시]
```
curl -X DELETE -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/tags/{tagId}
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

## UID 관리

### UID 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/uids|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Query parameter]

|값|	타입|	필수|	설명|
|---|---|---|---|
|wheres|	List:String|	X|	조회 조건.</br> 알파뱃, 숫자, 괄호로 이루어진 문자열 배열. </br>괄호는 1개, AND, OR은 3개까지 사용할 수 있다.</br> (예시) tagId1,AND,tagId2|
|offsetUid|	String|	X|	offset UID|
|offset|	Integer|	X| offset(Default : 0)|
|limit|	Integer|	X|	조회 건수(Default : 15)|

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/uids?wheres=tagId1,OR,tagId2&offset=0&limit=10"
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "data": {
            "uids": [
                {
                    "uid": String,
                    "tags": [
                        {
                            "tagId": String,
                            "tagName": String,
                            "createDate": String,
                            "updateDate": String
                        }
                    ],
                    "contacts": [
                        {
                            "contactType": String,
                            "contact": String,
                            "createDate": String"
                        }
                    ]
                }
            ],
            "isLast": Boolean,
            "totalCount": Integer
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	List|	데이터 영역|
|-- uids|	List|	UID 목록|
|--- uid | String  | UID |
|--- tags | List | 태그 정보 리스트 |
|---- tagId | String | 태그 ID |
|---- tagName | String | 태그 이름 |
|---- createDate | String | 태그 생성일시 |
|---- updateDate | String | 태그 수정일시 |
|--- contacts | List | 연락처 리스트 |
|---- contactType | String | 연락처 타입 (EMAIL_ADDRESS)|
|---- contact | String | 연락처 (메일 주소)) |
|---- createDate | String | 연락처 생성일시 |
|-- isLast | Boolean | 마지막 리스트 여부 |
|-- totalCount | Integer | 총 데이터 건수  |

### UID 단건 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/uids/{uid}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|uid|	String|	UID |

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/uids/{uid}"
```

#### 응답

```
{
    "header": {
        "isSuccessful": String,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "data": {
            "uid": "",
            "tags": [
                {
                    "tagId": String,
                    "tagName": String,
                    "createDate": String,
                    "updateDate": String
                }
            ],
            "contacts": [
                {
                    "contactType": String,
                    "contact": String,
                    "createDate": String
                }
            ]
        }
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	List|	데이터 영역|
|-- uid | String  | UID |
|-- tags | List | 태그 정보 리스트 |
|--- tagId | String | 태그 ID |
|--- tagName | String | 태그 이름 |
|--- createDate | String | 태그 생성일시 |
|--- updateDate | String | 태그 수정일시 |
|-- contacts | List | 연락처 리스트 |
|--- contactType | String | 연락처 타입 |
|--- contact | String | 연락처 (메일 주소)) |
|--- createDate | String | 연락처 생성일시 |

### UID 등록

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/uids|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Request body]

|값|	타입|	필수|	설명|
|---|---|---|---|
|uid|	String|	O|	UID|
|tagIds|	List:String|	O|	태그 ID 목록|
|contacts|	List|	O|	메일 주소 목록 |
|-contactType| String| O| 연락처 타입 |
|-contact| String| O| 연락처 (메일 주소) |

[주의]

* tagIds가 주어지는 경우 contacts는 필수 값이 아닙니다.
* contacts가 주어지는 경우 tagIds는 필수 값이 아닙니다.
* 본 상품의 경우, contactType은 반드시 "EMAIL_ADDRESS" 값으로 요청해야 합니다.

[예시]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/uids -d '{"uids":[{"uid":"sample-uid","tagIds":["tagId1"],"contacts":[{"contactType":"EMAIL_ADDRESS","contact":"customer1@nhn.com"},{"contactType":"EMAIL_ADDRESS","contact":"customer2@nhn.com"}]}]}'
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

### UID 삭제

#### 요청

[URL]

|Http method|	URI|
|---|---|
|DELETE|	/email/v1.4/appKeys/{appKey}/uids/{uid}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|uid|	String|	UID|

[예시]
```
curl -X DELETE -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/uids/{uid}
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

### 메일 주소 등록

#### 요청

[URL]

|Http method|	URI|
|---|---|
|POST|	/email/v1.4/appKeys/{appKey}/uids/{uid}/email-addresses|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|uid|	String|	UID|

[Request body]

|값|	타입|	필수|	설명|
|---|---|---|---|
|emailAddress|	String|	O|	메일 주소|

[예시]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/uids/{uid}/email-addresses -d '{"emailAddress" : "customer1@nhn.com"}'
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

### 메일 주소 삭제

#### 요청

[URL]

|Http method|	URI|
|---|---|
|DELETE|	/email/v1.4/appKeys/{appKey}/uids/{uid}/email-addresses/{emailAddress}|

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|
|uid|	String|	UID|
|emailAddress|	String|	메일 주소|

[예시]
```
curl -X DELETE -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/uids/{uid}/email-addresses/customer1@nhn.com
```

#### 응답

```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

## 통계 조회

### 통합 통계 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
|GET|	/email/v1.4/appKeys/{appKey}/statistics/view |

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Query parameter]

|값|	타입|	필수| 설명|
|---|---|---|---|
|from|	String|	O | 통계 조회 시작 날짜<br/> yyyy-mm-dd HH:mm|
|to|	String|	O | 통계 조회 종료 날짜<br/> yyyy-mm-dd HH:mm|
|searchType| String | O | 통계 구분<br/>DATE:날짜별, TIME:시간별, DAY:요일별 |
|mailTypes | String | X | 메일 타입<br/>NORMAL:일반, MASS:대량<br/>복수 입력 가능(mailTypes=NORMAL&mailTypes=MASS) |
|adYn | String | X | 광고 여부<br>Y:광고, N:광고 아님<br>입력하지 않으면 전체|
|templateId | String | X | 템플릿 ID |

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/statistics/view?from=2018-03-21+00%3A00&to=2018-03-23+00%3A00&searchType=DATE&mailTypes=NORMAL&adYn=Y&templateId=templateId1"
```

#### 응답

```json
{
    "isSuccessful": Boolean,
    "resultCode": Integer,
    "resultMessage": String
},
    "body" : {
        "data" : [
          {
                  "divisionName": String,
                  "requestedCount": long,
                  "sentCount": long,
                  "receivedCount": long,
                  "openedCount": long,
                  "sentRate": String,
                  "receivedRate": String,
                  "openedRate": String
        }
        ]
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- data|	List|	데이터 영역|
|-- divisionName | String | 통계 기준(날짜/시간/요일) |
|-- requestedCount | Long | 발송 요청 카운트 |
|-- sentCount | Long | 발송 카운트 |
|-- receivedCount | Long | 수신 카운트 |
|-- openedCount | Long | 오픈 카운트 |
|-- sentRate | String | 발송율 |
|-- receivedRate | String | 수신율 |
|-- openedRate | String | 오픈율 |

## 수신 거부 관리

### 수신 거부 조회

#### 요청

[URL]

|Http method|	URI|
|---|---|
| GET |	/email/v1.4/appKeys/{appKey}/block-receivers |

[Path parameter]

|값|	타입|	설명|
|---|---|---|
|appKey|	String|	고유의 appKey|

[Query parameter]

|값|	타입|	필수| 설명|
|---|---|---|---|
|mailAddress|	String|	X| 수신거부 목록에 등록되어 있는 이메일 주소|
|pageNum|	Integer|	X|	페이지 번호(Default : 1)|
|pageSize|	Integer|	X|	조회 건수(Default : 15)|

[예시]
```
curl -X GET -H "Content-Type: application/json;charset=UTF-8" "https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/block-receivers?mailAddress=customer1@nhn.com&pageNum=1&pageSize=10"
```

#### 응답
```
{
    "header": {
        "isSuccessful": Boolean,
        "resultCode": Integer,
        "resultMessage": String
    },
    "body": {
        "totalCount": Integer,
        "data": [
            {
                "mailAddress": String,
                "blockDate": String
            }
        ]
    }
}
```

|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|
|body|	Object|	본문 영역|
|- pageNum|	Integer|	현재 페이지 번호|
|-pageSize|	Integer|	조회된 데이터 건수|
|- totalCount|	Integer|	총 데이터 건수|
|- data|	List|	데이터 영역|
|-- mailAddress | String | 수신거부 이메일 주소 |
|-- blockDate | String | 수신거부 날짜 (yyyy-MM-dd HH:mm:ss.S)

### 수신 거부 등록

#### 요청

[URL]

|Http method|	URI|
|---|---|
| POST |	/email/v1.4/appKeys/{appKey}/block-receivers |

[Request body]

|값|	타입|	필수 | 설명|
|---|---|---|---|
| blockReceiverList |  ㅣList | O | 수신거부 리스트 |
| - mailAddress | String | O | 수신거부 이메일 주소 |
| - blockDate | String | X | 수신 거부 날짜 (yyyy-MM-dd HH:mm:ss) |

[예시]
```
curl -X POST -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/block-receivers -d '{"blockReceiverList":[{"mailAddress":"customer1@nhn.com","blockDate":"2018-03-01 00:00:00"}]}'
```

#### 응답
```
{
  "header": {
    "isSuccessful":  Boolean,
    "resultCode": Integer,
    "resultMessage": String
  }
}
```
|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|

### 수신 거부 삭제
#### 요청

[URL]

|Http method|	URI|
|---|---|
| PUT |	/email/v1.4/appKeys/{appKey}/block-receivers |

[Request body]

|값|	타입|	필수 | 설명|
|---|---|---|---|
| deleted | Boolean | O | 수신 거부 삭제를 명시하는 필드 |
| blockReceiverList |  ㅣList | O | 수신거부 리스트 |
| - mailAddress | String | O | 수신거부 이메일 주소 |

[예시]
```
curl -X PUT -H "Content-Type: application/json;charset=UTF-8" https://api-mail.cloud.toast.com/email/v1.4/appKeys/{appKey}/block-receivers -d '{"deleted":true,"blockReceiverList":[{"mailAddress":"customer1@nhn.com"}]}'
```

#### 응답
```
{
  "header": {
    "isSuccessful":  Boolean,
    "resultCode": Integer,
    "resultMessage": String
  }
}
```
|값|	타입|	설명|
|---|---|---|
|header|	Object|	헤더 영역|
|- isSuccessful|	Boolean|	성공여부|
|- resultCode|	Integer|	실패 코드|
|- resultMessage|	String|	실패 메시지|