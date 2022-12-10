---
title: "Swagger 파일 분리하여 관리하기"
date: 2019-12-22T16:36:37+09:00
tags: ["openapi", "documentation"]
aliases:
  - "/2019/12/swagger_file_split/"
cover:
  image: "https://raw.githubusercontent.com/swagger-api/swagger.io/wordpress/images/assets/SWU-logo-clr.png"
---

Swagger로 API 문서를 작성하다 보면 스팩 파일(swagger.yaml)이 너무 길어져서 관리가 어려울 때가 있습니다. 그래서 swagger에서는 `$ref`를 제공해 parameter와 ruqestbody, response 등을 모듈처럼 사용할 수 있도록 제공하지만 이것 또한 API가 많아지면 스펙파일이 길어지기 마련이죠. 그래서 parameter와 ruqestbody, response은 물론 엔드포인트들을 파일로 분리해서 모듈처럼 관리하여 쉽게 유지보수 하는 걸 이번 포스트에서 해보려고 합니다. 모든 개발에는 유지보수가 가장 중요한 법이니까요.

<!--more-->

## 준비하기

우선, 우리는 스펙파일을 준비해야 합니다. 이번 포스트는 openapi v3를 기준으로 작성할것이며 v2도 전체적인 흐름은 같으니 따라해보셔도 문제가 없습니다. 먼저, 이미 스펙파일을 가지고 계신 분은 가지고 계신 스펙파일을 이용하시면되지만 저는 [petstore.yaml](https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/examples/v3.0/petstore.yaml)을 조금 [수정한 파일](https://github.com/psi59/swagger_file_split_example/blob/master/edited_petstore.yaml)을 사용해 나눠보려 합니다.

```yaml
openapi: "3.0.0"
info:
  version: 1.0.0
  title: Swagger Petstore
  license:
    name: MIT
servers:
  - url: http://petstore.swagger.io/v1
paths:
  /pets:
    get:
      summary: List all pets
      operationId: listPets
      tags:
        - pets
      parameters:
        - name: limit
          in: query
          description: How many items to return at one time (max 100)
          required: false
          schema:
            type: integer
            format: int32
      responses:
        '200':
          description: A paged array of pets
          headers:
            x-next:
              description: A link to the next page of responses
              schema:
                type: string
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Pets"
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
    post:
      summary: Create a pet
      operationId: createPets
      tags:
        - pets
      requestBody:
        description: unexpected error
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                tags:
                  type: string
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
  /pets/{petId}:
    get:
      summary: Info for a specific pet
      operationId: showPetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Pet"
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"

    put:
      summary: Edit a pet
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      requestBody:
        description: unexpected error
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                tags:
                  type: string
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
components:
  schemas:
    Pet:
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        tags:
          type: string
    Pets:
      type: array
      items:
        $ref: "#/components/schemas/Pet"
    Error:
      required:
        - code
        - message
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
```

우선 `edited_petstore.yaml`파일을 보시면 한번이라도 swagger를 사용해보신 분이라면 이정도 스펙파일은 길지 않다는 걸 잘 알고 계실 겁니다.

이미 `$ref`를 사용해 `schema`들을 모듈화해서 중복을 최대한 줄이려고 했습니다만.
중복되는 부분이 세개가 보이네요.

1. `POST /pets`와 `PUT /pets/{pet_id}`의 `requestBody`
2. `GET /pets/{pet_id}`와 `PUT /pets/{pet_id}`의 `pet_id` 파라메터
3. 반복되는 `response`의 `unexpected error`

저는 이 두개의 중복과 스키마, 그리고 엔드포인트들을 파일로 나눠보려 합니다.

### 1. requestBody 분리하기

중복되는 `requestBody`를 `requestBodies/pet.yaml` 파일로 옮겨보겠습니다.

```yaml
# requestBodies/pet.yaml
description: 'pet create/edit request'
content:
  application/json:
    schema:
      type: object
      properties:
        name:
          type: string
        tags:
          type: string
```

```yaml
# swagger.yaml
/pets:
    post:
      summary: Create a pet
      operationId: createPets
      tags:
        - pets
      requestBody:
				$ref: 'requestBodies/pet.yaml'
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
  /pets/{petId}:
    put:
      summary: Edit a pet
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      requestBody:
        $ref: 'requestBodies/pet.yaml'
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
```

위 처럼 나눈 파일을 봤을때 주의해야할 것이 `requestBody`에 있는 `$ref`입니다.
`$ref`는 `swagger.yaml`의 위치를 기준으로 relative path로 `pet.yaml` 파일을 찾습니다.

```bash
.
├── requestBodies
│   └── pet.yaml
└── swagger.yaml
```

파일을 나눈 후 위와같은 폴더 구조를 갖습니다.

### 2. parameter 분리하기

이번에는 중복되는 parameter인 `petId`를 분리 해보겠습니다.

```yaml
# parameter.yaml
queryPetId:
  name: petId
  in: path
  required: true
  description: The id of the pet to retrieve
  schema:
    type: string
```

```yaml
# swagger.yaml
/pets/{petId}:
    get:
      summary: Info for a specific pet
      operationId: showPetById
      tags:
        - pets
      parameters:
        - $ref: 'parameters.yaml#/queryPetId'
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Pet"
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"

    put:
      summary: Edit a pet
      tags:
        - pets
      parameters:
        - $ref: 'parameters.yaml#/queryPetId'
      requestBody:
        $ref: 'requestBodies/pet.yaml'
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
```

위와 같이 파일을 분리하였고 아래와 같은 폴더 구조가 되었습니다.

```bash
.
├── parameters.yaml
├── requestBodies
│   └── pet.yaml
└── swagger.yaml
```

```yaml
parameters:
	- $ref: 'parameters.yaml#/queryPetId'
```

이번 `$ref`는 좀 다르게 생겼습니다. 파일을 링크하긴 하지만 뒤에 `#/queryPetId`가 붙었습니다.
이 표현은 한 파일 안에 다수의 파라메터들이 적용되었을 때 특정 파라메터를 참조하고 싶을 때 사용할 수 있는 표현입니다. 이 표현은 `parameter`에만 한정된 것이 아니고 `requestBody`, `response`, `schema`에도 사용할 수 있습니다.

### 3. 중복된 response 분리하기

이번에는 `response`에서 계속 반복되던 `unexpect error` 부분을 수정해보겠습니다.
이 때 `components/schemas`에 있는 `schema`들도 함께 파일로 분리해보겠습니다.

```yaml
# responses/unexpected_error.yaml
description: unexpected error
content:
  application/json:
    schema:
      $ref: "#/components/schemas/Error"
```

먼저 손쉽게 error를 분리했습니다.

```yaml
# schemas.yaml
Pet:
  required:
    - id
    - name
  properties:
    id:
      type: integer
      format: int64
    name:
      type: string
    tags:
      type: string
Pets:
  type: array
  items:
    $ref: "schemas.yaml#/Pet"
Error:
  required:
    - code
    - message
  properties:
    code:
      type: integer
      format: int32
    message:
      type: string
```

그다음 위와 같이 `schemas.yaml`을 분리하고 `components`를 삭제했습니다. 이때 `Error`를 찾지 못해서 에러가 발생합니다. 이유는 `components/schemas`들을 모두 삭제해서 찾을 수 없기 때문이죠.
이때 파일간의 참조는 어떻게 하는지 배울 수 있습니다.  

분리한 `# responses/unexpected_error.yaml` 아래와 같이 수정하겠습니다.

```yaml
# responses/unexpected_error.yaml
description: unexpected error
content:
  application/json:
    schema:
      $ref: "../schemas.yaml#/Error"
```

수정한 파일 내용을 보면 `swagger.yaml`가 다른 파일을 참조할 때처럼 
`responses/unexpected_error.yaml`을 기준으로  `schemas.yaml`을 참조하시면 됩니다.  

위와 같이 수정하고 나면 최종적으로 `swagger.yaml`는 아래와 같은 형태를 띄게 됩니다.

```yaml
# swagger.yaml
openapi: "3.0.0"
info:
  version: 1.0.0
  title: Swagger Petstore
  license:
    name: MIT
servers:
  - url: http://petstore.swagger.io/v1
paths:
  /pets:
    get:
      summary: List all pets
      operationId: listPets
      tags:
        - pets
      parameters:
        - name: limit
          in: query
          description: How many items to return at one time (max 100)
          required: false
          schema:
            type: integer
            format: int32
      responses:
        '200':
          description: A paged array of pets
          content:
            application/json:
              schema:
                $ref: "schemas.yaml#/Pets"
        default:
          $ref: 'responses/unexpected_error.yaml'
    post:
      summary: Create a pet
      operationId: createPets
      tags:
        - pets
      requestBody:
        $ref: 'requestBodies/pet.yaml'
      responses:
        '201':
          description: Null response
        default:
          $ref: 'responses/unexpected_error.yaml'
  /pets/{petId}:
    get:
      summary: Info for a specific pet
      operationId: showPetById
      tags:
        - pets
      parameters:
        - $ref: 'parameters.yaml#/queryPetId'
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: "schemas.yaml#/Pet"
        default:
          $ref: 'responses/unexpected_error.yaml'

    put:
      summary: Edit a pet
      tags:
        - pets
      parameters:
        - $ref: 'parameters.yaml#/queryPetId'
      requestBody:
        $ref: 'requestBodies/pet.yaml'
      responses:
        '201':
          description: Null response
        default:
          $ref: 'responses/unexpected_error.yaml'
```

그리고 아래와 같은 형태의 디렉토리 구조가 됩니다.

```bash
.
├── parameters.yaml
├── requestBodies
│   └── pet.yaml
├── responses
│   └── unexpected_error.yaml
├── schemas.yaml
└── swagger.yaml
```



## 요약

- `$ref`를 이용하여 스펙파일을 모듈화하면 유지보수가 용이해진다.
- 참조
	- 파일 참조는 `$ref`를 사용한다.
	- 파일간의 참조는 현재 파일 위치를 기준으로 참조할 파일의 relative path를 사용하면 된다.



## 마치며...

 위의 예제들을 보면 두서없이 파일을 나눈것 처럼 보입니다. 이것은 단지 독자들에게 다양한 형태의 참조를 보여드리기 위해서 이런 형태를 만들어 보았습니다. 물론 실제로 사용할 때는 좀 더 일관성 있는 형태의 구조로 사용하는 것이 좋습니다.

 위의 예제들은 [이곳](https://github.com/psi59/swagger_file_split_example)에 업로드 했으니 참조하시면 될 것 같습니다.

### 추가적인 팁
저는 이렇게 만든 파일들을 배포할 때는 전체 파일을 배포하기보다는 하나의 파일로 묶어서 배포합니다. 
이때 사용할 수 있는게 `openapi-generator`입니다.  

```bash
openapi-generator generate -i ./swagger.yaml -g openapi-yaml -o compiled
```

위의 커맨드를 사용하게 되면 나눠져 있던 파일들을 모아서 우리가 처음에 나누지 않았던 형태의 파일처럼 하나로 합쳐주게 됩니다. 

자세한 설치방법과 사용법은 (openapi-generator)[https://github.com/OpenAPITools/openapi-generator] 를 참조하시면 될 것 같습니다.



## Reference

- https://swagger.io/docs/specification/using-ref


