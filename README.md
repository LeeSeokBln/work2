아테나 쿼리로 index.html 만들어서 s3 저장하기
```
import boto3
import time

def lambda_handler(event, context):
    # Athena에서 쿼리 결과 가져오기
    results = query_athena()

    # 결과를 기반으로 HTML 생성
    html_content = generate_html(results)

    # HTML 파일을 S3에 저장
    save_to_s3(html_content)

    return {
        'statusCode': 200,
        'body': 'HTML generated and saved to S3 successfully!'
    }

def query_athena():
    athena_client = boto3.client('athena')
    query = """
    SELECT 
        CASE 
            WHEN response LIKE '2%' THEN '2xx'
            WHEN response LIKE '4%' THEN '4xx'
            WHEN response LIKE '5%' THEN '5xx'
        END AS response_category,
        COUNT(*) as count
    FROM logs
    WHERE response LIKE '2%' OR response LIKE '4%' OR response LIKE '5%'
    GROUP BY 
        CASE 
            WHEN response LIKE '2%' THEN '2xx'
            WHEN response LIKE '4%' THEN '4xx'
            WHEN response LIKE '5%' THEN '5xx'
        END;
    """
    response = athena_client.start_query_execution(
        QueryString=query,
        ResultConfiguration={
            'OutputLocation': 's3://your_bucket_name_here/'  # 여기에 S3 버킷 이름을 입력하세요.
        }
    )
    query_execution_id = response['QueryExecutionId']

    while True:
        status = athena_client.get_query_execution(QueryExecutionId=query_execution_id)['QueryExecution']['Status']['State']
        if status in ['SUCCEEDED', 'FAILED', 'CANCELLED']:
            break
        time.sleep(5)  # 5초마다 쿼리 상태 확인

    if status != 'SUCCEEDED':
        raise Exception(f"Athena query failed with status: {status}")

    result = athena_client.get_query_results(QueryExecutionId=query_execution_id)
    rows = result['ResultSet']['Rows']
    data = {}
    for row in rows[1:]:  # 첫 번째 행은 헤더이므로 제외
        category = row['Data'][0]['VarCharValue']
        count = int(row['Data'][1]['VarCharValue'])
        data[category] = count

    return data

def generate_html(results):
    if results.get('5xx', 0) >= 10000 or results.get('4xx', 0) >= 5000:
        status = '나쁨'
    elif results.get('4xx', 0) > 0 or results.get('5xx', 0) > 0:
        status = '보통'
    else:
        status = '좋음'

    html_content = f"""
    <html>
    <head><title>Status Report</title></head>
    <body>
        <h1>Status: {status}</h1>
        <p>2xx Responses: {results.get('2xx', 0)}</p>
        <p>4xx Responses: {results.get('4xx', 0)}</p>
        <p>5xx Responses: {results.get('5xx', 0)}</p>
    </body>
    </html>
    """
    return html_content

def save_to_s3(html_content):
    s3 = boto3.client('s3')
    bucket_name = 'your_bucket_name_here'  # 여기에 S3 버킷 이름을 입력하세요.
    s3.put_object(Bucket=bucket_name, Key='index.html', Body=html_content, ContentType='text/html')

```
