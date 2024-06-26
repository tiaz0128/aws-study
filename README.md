# AWS Study

AWS 를 사용하는 3가지 방법으로 공부한다. services 폴더에 서비스별로 공부한 내용을 정리한다.

1. AWS console
    - `console.md` 파일에 내용을 정리
2. awscli
    - `cli.md` 파일에 내용을 정리
    - [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest)
3. boto3(python)
    - `script.py` 파일에 내용을 정리
    - [Boto3 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
    - [AWS SDK Code Examples](https://github.com/awsdocs/aws-doc-sdk-examples)

## 임시 자격 증명(Assume-Role)

AWS 관련 정보는 유출시 치명적이므로, 반드시 임시 자격 증명(Assume-Role)을 통해서 최소한의 권한을 설정 테스트를 수행한다.

## aws-cli 설치

```bash
$ apt update

$ apt install awscli jq
```

## aws-cli configure

```bash
$ aws configure
```

```text
AWS Access Key ID [None] :
AWS Secret Access Key [None] :
Default region name [None] : ap-northeast-2
Default output format [None] : None
```

## aws-cli Assume-Role

```bash
$ aws sts assume-role --role-arn <ROLE_ARN> --role-session-name <test-session>
```

```text
{
    "Credentials": {
        "AccessKeyId": "ASIA.......",
        "SecretAccessKey": "YwyGY8dF5.......",
        "SessionToken": "IQoJb3JpZ2.......",
        "Expiration": "2024-04-21T08:08:12Z"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROA6Q5UMFQN.......",
        "Arn": "arn:aws:......."
    }
}
```

발급 받은 임시 자격 증명(Assume-Role) `Credentials` 토큰을 환경 변수로 세팅

- `Credentials.AccessKeyId`
- `Credentials.SecretAccessKey`
- `Credentials.SessionToken`

```bash
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_SESSION_TOKEN=
```

## aws-cli Assume-Role 스크립트

위의 작업을 bash 스크립트로 작성해두고 실행하자.

```bash
$ export ROLE_ARN=

$ source ./scripts/set_assume_role.sh
```

```bash
$ aws sts get-caller-identity
```

## boto3 Assume-Role

- `.env` 파일에 필요한 정보를 입력
- pytest fixture 로 임시 자격 증명(Assume-Role) `Credentials` 토큰을 발급 받아서 테스트

```text
AWS_ACCOUNT_ID=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
REGION_NAME=ap-northeast-2

ROLE_ARN=
```

```python
def assume_role(aws_user_session: Session, role_arn):
    sts_client = aws_user_session.client("sts")

    response = sts_client.assume_role(
        RoleArn=role_arn, RoleSessionName="boto3-assume-session"
    )

    return response["Credentials"]
```
