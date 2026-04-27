# Account Status

SFDC 정보를 이용해 Account의 현황을 정리해서 보고서로 작성합니다.

## Workflow

### Step 1) Resolve target account from account name

사용자에게 계정명을 받습니다. 이름으로 계정을 조회해 정확히 1개 대상 계정을 확정합니다.

### Step 2) Fetch financial and mapping data

확정된 단일 계정에 대해 아래 데이터를 조회합니다.

- spend summary
- spend history (월별 포함)
- AWS account mappings (chargeR12 기준 정렬, 충분한 limit)
- 서비스별 지출 데이터(서비스명 + R12 금액이 포함된 breakdown; 가능 시 chargeR12 기준 정렬)

### Step 3) Compute metrics

아래 지표를 계산합니다.

- Rolling 12개월 성장률
- 전년 동월(YoY) 성장률
- 최근 3개월 추이(MoM)
- AWS 상위 10 계정의 chargeR12 합계 및 비중
- AWS 계정별 전체 비중(필수):
- 서비스별 R12 상위 10 합계 및 비중
- 서비스별 전체 비중(필수):

### Step 4) Build single-account HTML report

리포트 파일을 HTML로 작성합니다.


### Step 7) Ask recipient email and send immediately

보고서(HTML + 차트 이미지)가 완성되면 수신 이메일 주소를 사용자에게 묻습니다. 

### Step 8) Send email

차트 이미지를 이메일로 보낼수 있도록 축소하여 발송합니다.


## 설치

### NPM 설치

```bash
npm install
```


### Amazon 인증

midway에 패스워드 넣고 인증합니다. (FIDO2)

```bash
mwinit -f
```

FIDO2가 아닌 경우에 아래 방식으로 합니다.

```bash
mwinit -o -s
```

Toolbox 설치를 합니다.

```bash
toolbox install toolbox
```

### aws_sentral

Sentral MCP을 통해 SFDC 정보를 조회합니다.

```java
{
   "mcpServers":{
      "aws_sentral":{
         "command":"~/.toolbox/bin/aws-sentral-mcp",
         "args":[
            
         ]
      }
   }
}
```

### aws_outlook

Outlook MCP을 이용해 메일을 발송합니다.

```java
{
   "mcpServers":{
      "aws_outlook":{
         "command":"~/.toolbox/bin/aws-outlook-mcp",
         "args":[
            
         ],
         "env":{
            "OUTLOOK_MCP_ENABLE_WRITES":"true"
         }
      }
   }
}
```




