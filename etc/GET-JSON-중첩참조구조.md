# LogCollector 데이터 바인딩 방식 변경

## 문제

### 내부 로직 문제

- query string으로 `items[0][id]`, `items[0][count]` 와 같이 대괄호 쌍이 2개인 데이터가 들어오는 경우가 있어 아래와 같이 ParameterMap에서 텍스트로 읽고 파싱해서 처리

    ```javascript
    parameterMap.forEach(
            (k, v) -> {
                String dotKey = k.replaceAll("\\[(\\D+)", ".$1")
                        .replaceAll("(\\.\\w+)]", "$1");
                userLogDtoProps.addPropertyValue(dotKey, v);
            }
    );
    ```

- 이 방식으로는 대괄호 쌍이 2개인 경우만 처리가능한데, `items[0][options][0][stck_no]` 처럼 대괄호 쌍이 3개 이상인 데이터 처리 필요


### 로그 전송 방식 문제

- 로그 내용이 담긴 JSON을 POST로 서버에 보내면, 스프링 기반의 서버에서는 컨트롤러의 파라미터에서 RequestBody를 붙여서 DTO에 쉽게 바인딩 할 수 있으나,
- 현재로는 GET과 POST가 혼재되어 있고, 대부분 GET으로 들어오고 있음
  - GET으로 들어오는 이유는 Cross Domain Request를 처리하기 위해 JSONP를 사용했기 때문인 것으로 추정
  - 이미 다수의 클라이언트에 GET으로 전송하도록 구현되어 있으므로 POST로 전환하는 것은 현실적으로 불가능
- GET으로 들어올 경우 아래와 같은 요청은

    ```javascript
    $.ajax({
        url: '어쩌구-서버-API',
        contentType: 'application/json',
        method: 'GET',
        crossDomain: true,
        data: {
            id: "321",
            items: [
                {
                    id: "abc987",
                    count: 3
                }
            ],
            emails: ['abc@abc.com', 'sdf@sdf.com']
        }
    }).done(function() {
        // 성공 시 처리
    });
    ```

- 실제 다음과 같은 HTTP요청으로 서버에 들어옴(알아보기 위해 URL Decoding을 적용, 실제로는 URL Encoding 된 채로 들어옴)

    ```
    GET /어쩌구-서버-API?id=321&items[0][id]=abc987&items[0][count]=3&emails[]=abc@abc.com&emails[]=sdf@sdf.com
    ```

- 따라서 `items[0][options][0][stck_no]` 처럼 대괄호 쌍이 3개 이상인 key로 들어오는 데이터도 핸들링 할 수 있는 로직 필요
 

## 해결

### query string 파싱이 아닌 데이터 바인딩 방식으로 전환

- 기존처럼 request parameter의 개별 key를 하나하나 분해해서 `indexAndKey[0]`, `indexAndKey[1]` 같은 방식으로 처리하는 것은 확장성이 부족하며 로직이 불필요하게 복잡해짐
- 따라서 기존의 분해 방식 대신 스프링의 `MutablePropertyValues`와 `DataBinder`를 활용해서 DTO에 바인딩하는 방식으로 전환
  - 정규표현식을 사용해서 `items[0][options][0][stck_no]` 를 `items[0].options[0].stck_no` 로 변환해서 `MutablePropertyValues`에 넣고, `DataBinder`로 DTO에 바인딩

        ```java
        public UserLogDTO convertRequestParameterMapIntoUserLogDTO(Map<String, String[]> parameterMap) {
    
            final MutablePropertyValues userLogDtoProps = new MutablePropertyValues();
    
            parameterMap.forEach(
                    (k, vArray) -> {
                        String dotKey =
                                k.replaceAll("\\[]", "")
                                        .replaceAll("\\[(\\D+)", ".$1")
                                        .replaceAll("]\\[(\\D)", ".$1")
                                        .replaceAll("(\\.\\w+)]", "$1");
    
                        // 숫자로 시작하는 key도 받아주는 버전
    //                                k
    //                                        .replaceAll("\\['", "[")
    //                                        .replaceAll("']", "]")
    //                                        .replaceAll("\\[\"", "[")
    //                                        .replaceAll("\"]", "]")
    //                                        .replaceAll("\\[]", "")
    //                                        .replaceAll("\\[(\\d+)]", "!#%@$1%@#!")
    //                                        .replaceAll("\\[", ".")
    //                                        .replaceAll("]", "")
    //                                        .replaceAll("!#%@(\\d+)%@#!", "[$1]");
                        userLogDtoProps.addPropertyValue(dotKey, vArray);
                    }
            );
    
            UserLogDTO userLogDTO = new UserLogDTO();
            DataBinder binder = new DataBinder(userLogDTO);
            binder.bind(userLogDtoProps);
    
            // 아래와 같이 userLogDTO.getItems(), "items", TypeReference<UserActionLog.Item[]>, userLogDTO::setter를
            // 하드 코딩성으로 지정하는 것보다 Controller에서 주입받아 처리하는 것이 더 나은 방식이나
            // 그렇게 하려면 메서드 레퍼런스 등의 처리가 너무 복잡해져서 이 정도로 타협
            // 추후 items와 user외에 객체형 데이터가 'another[key]=value'가 아니라 'another={"key":"value"}'와 같이 추가된다면 그때 고려
            if (userLogDTO.getItems() == null) {
                setObjectsByString(parameterMap, "items", new TypeReference<UserActionLog.Item[]>() {}, userLogDTO::setItems);
            }
            if (userLogDTO.getUser() == null) {
                setObjectsByString(parameterMap, "user", new TypeReference<UserActionLog.User>() {}, userLogDTO::setUser);
            }
    
            return userLogDTO;
        }
        ```

  - [https://homoefficio.github.io/2017/04/25/Spring-가-포함된-URL-파라미터-바인딩-하기/](https://homoefficio.github.io/2017/04/25/Spring-가-포함된-URL-파라미터-바인딩-하기/) 참고

### 장점

- DTO를 활용한 데이터 바인딩 방식을 사용하면 추후 items 메타 정보의 로그 항목이 추가되어도, 별도의 하드코딩 없이 DTO에만 추가하면 되므로 확장성과 유지보수성이 좋아짐
- 다만 로그 요청이 `items[0][id]` 등의 형식으로만 들어오는 것이 아니라, `items=[{"id":""abc", options:{"stck_no":"111", ...}, ...}]` 와 같이 items를 key로 하고 나머지 내용은 모두 하나의 문자열 값으로 들어오는 케이스가 있어서,
  - items 나 user 내부에 추가되는 로그 항목은 = 의 오른쪽에 문자열 값으로 들어가므로 DTO에서만 추가하면 됨 
  - 관련 코드는 UserLogDTOConverter.convertRequestParameterMapIntoUserLogDTO() 의 하단에 관련 로직과 주석 참고

## 기타

### 스프링 기능으로 Pull Request 신청

- https://jira.spring.io/browse/SPR-15492 로 신청했지만 거절
