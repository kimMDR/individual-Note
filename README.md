![image](https://github.com/user-attachments/assets/0427cd21-b830-4800-8937-1c5e1b671fe3)멘토님이 주신 자료 중에서 github에 어떤걸 클론해서 실습해볼 수 있는지 모르겠길래 옆건물 혜수네 노트정리 보면서 따라해봤습니다

(+혜수 짱이다)

- 자자 elk환경 구축하다가 오류 뒤1지게 많이 났던 `vm.max_map_count` 문제
    
    문제 상황
    
    - Docker Compose로 ELK 스택 실행 시 Kibana 컨테이너가 `unhealthy` 상태가 되어 실행 실패
    - Elasticsearch 로그 확인 결과 아래 오류 메시지 발견
    
    ```
    bootstrap check failure [1] of [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    ```
    
    원인
    
    - Elasticsearch는 많은 메모리 맵 영역이 필요해 `vm.max_map_count` 커널 파라미터를 최소 262144 이상으로 설정해야 함
    - 기본값(65530)이 너무 낮아 부팅 체크를 실패하여 컨테이너가 정상 실행되지 않음
    
    문제를 해결해보자
    
    1. 현재 `vm.max_map_count` 값 확인 (선택사항)
    
    ```bash
    sysctl vm.max_map_count
    ```
    
    2. `vm.max_map_count` 값을 262144로 일시 변경
    
    ```bash
    sudo sysctl -w vm.max_map_count=262144
    ```
    
    3. 설정을 영구 적용하기 위해 `/etc/sysctl.conf` 파일 편집
    
    ```bash
    sudo nano /etc/sysctl.conf
    ```
    
    - 파일 맨 아래에 다음 내용 추가
    
    ```
    vm.max_map_count=262144
    ```
    
    - 저장 `Ctrl + X` 종료)
    
    4. 변경된 설정 적용
    
    ```bash
    sudo sysctl -p
    ```
    
    5. Docker Compose로 ELK 스택 재시작
    
    ```bash
    sudo docker-compose down
    sudo docker-compose up -d
    ```
    

## Kibana 입성하기

멘토님 깃허브 들어가보면 `sudo docker-compose up -d` 명령어로 docker compose할 수 있다고 멘토님 github에 친절하게 쓰여있음 음하학



아래 명령어로 github clone하는 중

```bash
git clone https://github.com/gunh0/whs-utils.git
```

깃클론 끝나면 아래 경로로 들어가서 docker-compose할거임
```jsx
cd ~/ELK/whs-utils/simple_elk_for_siem/basic-elk
```

localhost:5601으로 들어와서 로그인 완료함

```php
ID: elastic

PW: p@ssw0rd1234
```

elk들어와서 Explore on my own 버튼을 누를건데 이건 
자동 데이터 수집 설정 없이 수동으로 Kibana 탐색을 시작하겠다는 의미임

Kibana 입성하면 뜨는 메뉴 설명~

**Enterprise Search** : 기업 맞춤형 검색 기능을 만들 수 있음

**Observability** : 시스템 운영 상태(서버 로그, 성능 등)를 모니터링하는거

**Security** : 보안 위협을 탐지하고 대응하는 데 쓰임 (SIEM 용도)

**Analytics** : 데이터를 직접 보고, 그래프나 표로 분석할 수 있음

## sample web logs 추가하기

메인(홈)에서 보이는 Try sample data를 눌러서 들어가봅니다


other sample data sets을 누르면 다른 다양한 데이터 추가 가능


들어오면 선택지 세 개가 보이는데 이 중에서 Sample seb logs를  install 할거임


## 메뉴에서 Discover들어가서 logs확인

Sample Web logs를 추가한 뒤 메뉴바에서 Discover로 들어가보면 

로그들을 확인할 수 있습니다

도대체 이 로그들이 뭘 찍어내서 보여주는건지 궁금해서 뭐냐면

| 필드명 | 의미 |
| --- | --- |
| **`@timestamp`** | 로그가 발생한 **정확한 시간**을 ISO 형식으로 나타냄 
(`2025-05-08T21:35:26.611Z` 등) |
| **`agent`** | 사용자 브라우저의 **User-Agent 정보** 
(예: `Mozilla/5.0`, `Chrome/11.0`) |
| **`bytes`** | 해당 요청에 의해 전송된 **데이터 크기 (바이트 단위)** |
| **`clientip`** | 접속한 **클라이언트의 IP 주소** |
| **`event.dataset`** | 이벤트가 속한 데이터셋 이름 (`sample_web_logs`) |
| **`extension`** | 웹 요청의 **파일 확장자** (예: `.html`, `.php`, `.jpg`) |
| **`geo.src`** | 요청한 IP의 **출발 국가 (source)** |
| **`geo.dest`** | 목적지 국가 (예: 서버 위치) |
| **`geo.coordinates`** | 요청자의 **위도/경도 좌표** (`POINT(위도 경도)` 형식) |
| **`host`** | 요청한 **서버 호스트 이름** |
| **`hour_of_day`** | 요청이 발생한 시각 (0~23) 숫자값 |

첫 번째 로그를 분석해보자

```json
{
  "@timestamp": "2025-05-08T21:35:26.611Z",
  "agent": "Mozilla/5.0 (X11; Linux x86_64)",
  "clientip": "87.60.55.198",
  "geo.src": "DE",
  "geo.dest": "US",
  "extension": "html"
}
```

> 2025년 5월 8일 21시 35분 26초에,
> 
> 
> 독일(DE)에 있는 IP `87.60.55.198` 사용자가
> 
> `Mozilla` 브라우저로 `.html` 페이지를 요청했고,
> 
> 서버는 미국(US)에 있었음
> 

## 메뉴에서 Dashboard 들어가보기

그럼 이렇게 web Traffic을 확인할 수 있어요. 들어가보면,

| 위치 | 의미 |
| --- | --- |
| **1613 Visits** | 지난 7일 동안 총 1,613번의 웹 요청 
발생 |
| **814 Unique Visitors** | 서로 다른 IP 기준으로 814명의 방문자 |
| **3.9% HTTP 4xx** | 전체 요청 중 3.9%가 클라이언트 오류 (예: 404 Not Found) |
| **3.4% HTTP 5xx** | 서버 오류 응답 (예: 500 Internal Error) 비율 |
| **Response Codes Over Time** | 시간대별로 2xx, 4xx, 5xx 응답 비율 추이 |
| **Total Requests and Bytes 
(지도)** | 요청 수와 위치(지리정보)를 시각화한 지도 |
| **Machine OS and Destination Sankey Chart** | 어떤 운영체제(iOS, Windows 등)에서 어디 국가로 접속했는지 흐름도 |

```json
샘플 웹 로그를 기반으로 
방문 수, 응답 코드, 사용자 IP, 운영체제, 바이트 전송량 등을
웹 서버에 일어난 활동을 시간대별, 통계적으로 시각화한 상황임
(이 웹사이트에 누가,얼마나,어떻게 접속했는가?를 확인할 수 있음)
자동 새로고침 기능도 있다고 합니다 *실시간 모니터링 가능
```

## 메뉴에서 Visualize Library 들어가기

들어와서 오른쪽 상단에 있는 create visualization 눌러서 만들어보자

Lens클릭

(drag&drop으로 간단하게 내가 원하는 필드를 시각화 할 수 있는 메뉴)


들어와서 Bar certical stacked → Area로 변경


`Horizontal axis`칸에 ← `@timestamp` 필드 삽입

`Vertical axis`칸에 ← `count` 삽입

[시각화]

오른쪽 상단에 있는 Save를 누르고 타이틀 입력해서 시각화 생성해봤음

## 이번에는 (https://localhost:9200)에 접속해볼게용

그냥 접속하려고 하니까 방화벽에서 막히길래 설정 들어가서 기본 보호 → 끄기로 바뀐 뒤 접속했음


사이트에 입성하려면 로그인 필요

ID: `elastic` 

PW: `p@ssw0rd1234` 

(https://localhost:9200)입성


kibana에서 Elasticsearch에 명령을 직접 날릴 수 있다고 하네요 해봅시다

기본 kibana 메인 화면에서 `Management`의 `Dev Tools`로 들어가


들어오면 Console에 명령어 입력이 가능한데 이거 입력할거

해당 명령어를 통해서 Elasitcsearch에 추가된 인덱스 목록을 확인할 수

있다고 합니다

```bash
GET _cat/indices?v
# ?v는 컬럼명을 함께 출력시키기 위한 설정
# 명령어 치고 난 다음에 request버튼 눌러서 실행
```


```bash
	*기존에 써있던 명령어
	#Click the variables button, above, to create your own variables.
	GET ${exampleVariable1} // _search
	{
		"query": {
			"${exampleVariable2}": {} // match_all
		}
	}
```

나는 왤캐 찔끔 뜨냐


인덱스 값 말고도 모든 값을 보고싶을 때는 아래와 같이 치면 된다고 합니

```bash
GET _search
{
  "query": {
    "match_all": {}
  }
}
```
