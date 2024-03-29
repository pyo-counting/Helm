Helm 3는 패키지 배포를 위해 OCI를 지원하는 container registory를 사용할 수 있다. Helm v3.8.0부터 OCI 지원은 기본적으로 활성화된다.

## OCI support prior to v3.8.0

### Enabling OCI support prior to v3.8.0

### OCI feature deprecation and behavior changes with v3.8.0

### OCI feature deprecation and behavior changes with v3.7.0

## Using an OCI-based registry

### Helm repositories in OCI-based registries

### Use hosted registries

## Commands for working with registries
### The `registry` subcommand
`helm registry` 명령어를 사용해 로그인(`helm registry login`), 로그아웃(`helm registry logout`) 가능하다.

### The `push` subcommand
`helm push` 명령어는 `helm package` 명령어를 통해 생성된 .tgz 파일에 대해서만 사용가능하다.

OCI registry에 업로드 시, `oci://` 접두사를 사용해야하며 chart 이름과 tag를 포함하면 안된다.

생성한 provenance file(`.prov`) 파일이 .tgz 파일과 같이 있을 떄, `helm push` 명령어에 의해 자동으로 업로드 된다. 명령어의 결과로 helm chart manifest에 layer가 생성된다.

### Other subcommands
아래 명령어 목록은 oci:// 프로토콜에 대해 사용 가능하다.

- helm push
- helm pull
- helm show
- helm template
- helm install
- helm upgrade

`helm push` 명령어를 제외한 명령어들은 이름을 명시해야 한다.

## Specifying dependencies
helm dependency update 명령어를 사용해 종속 chart를 다운로드할 수 있다.

``` yaml
dependencies:
  - name: mychart
    version: "2.7.0"
    repository: "oci://localhost:5000/myrepo"
```

## Helm chart manifest

## Migrating from chart repos