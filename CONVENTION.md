## Node.js 버전

기본적으로 LTS 버전 사용을 권장합니다.

.nvmrc에 명시된 버전을 기본으로 사용합니다.

```
nvm use
```

현재 Node.js의 LTS 버전은 22.12.0입니다.
 
혹시 현재 버전으로 고정시키고 싶으신 경우 다음과 같은 명령어를 통해 LTS 버전으로 Node.js 버전을 맞춰주세요

```
nvm alias default 22.12.0
```

volta 같은 도구를 사용하면 조금 더 엄격하게 관리할 수 있지만 volta 자체가 러닝커브가 될 것 같아.. 개개인이 쪼끔 신경을 써주는 방식으로 하면 좋을 것 같습니당



## Biome Setting

vscode를 사용하시는 경우 extension 에서 Biomejs를 설치해주세요 

https://biomejs.dev/reference/vscode

Biome는 prettier, eslint의 역할을 대체하는 포매터 겸 린터입니다.

Biome를 포매터로 사용하기 위한 설정은 .vscode폴더에 이미 되어있기 때문에 따로 신경을 쓰지는 않으셔도 됩니다!

익스텐션을 설치하셨다면 프로젝트를 껐다 켠 후 .ts 파일이나 .tsx 파일을 아무거나 수정해보세요!

저장했을떄 포매팅이 잘 동작한다면 설정이 된 것입니다.

### 자동 린팅, 포매팅

커밋, 푸시를 수행할 때 사용하지 않은 임포트, 변수들과 컨벤션에 맞지않는 코드를 자동으로 포매팅하는 과정에서 코드에 변경이 생깁니다.


## Convention

### Commit

기본적으로 conventional commit을 준수합니다.

커밋을 시도하실 때 만약 컨벤셔널 커밋을 준수하지 않은 경우 커밋이 실패하게됩니다.

https://www.conventionalcommits.org/ko/v1.0.0

컨벤셔널 커밋에 대한 정보는 위 링크에서 확인하실 수 있습니다.


### File Convetion

파일명은 파스칼 케이스 혹은 카멜 케이스만 허용합니다.

```
hello.ts // ok
helloWorld.ts // ok
hello-world.ts // X
```

이 규칙은 준수하지 않은 경우 커밋을 실패합니다.

## packages 

### @repo/http

데이터 페칭 코드가 필요한경우 @repo/http를 이용합니다.

```tsx
import { http } from "@repo/http";
export const getData = async () => {
    return http.get<리스폰스>('/hello')
}
```

get이외의 데이터는 다음과 같이 제네릭을 전달합니다.

```tsx
import { http } from "@repo/http";
export const postData = async () => {
    return http.post<리퀘스트, 리스폰스>()
}
```

### @repo/form

react-hook-form을 래핑한 "@repo/form"의 컴포넌트들을 사용합니다.

각 컴포넌트들은 헤드리스로 작성되어있으며 스타일은 따로 작성하도록 합니다. 예시 코드는 다음과 같습니다.

```tsx
import { zodResolver } from "@hookform/resolvers/zod";
import { FormControl, FormField, FormItem, FormLabel, FormMessage, FormProvider } from "@repo/form";
import { http } from "@repo/http";
import { useMutation, useSuspenseQuery } from "@tanstack/react-query";
import { useForm } from "react-hook-form";
import { z } from "zod";

const loginSchema = z.object({
  email: z.string({ message: "Email is required" }).email({ message: "Invalid email" }),
  password: z
    .string({ message: "Password is required" })
    .min(8, { message: "Password must be at least 8 characters" }),
});

type FormState = z.infer<typeof loginSchema>;

const usePostLogin = () => {
  return useMutation({
    mutationFn: (data: FormState) => http.post<FormState>("/login", data),
  });
};

export const Example = () => {
  const form = useForm<FormState>({
    resolver: zodResolver(loginSchema),
  });
  const { mutate, isPending } = usePostLogin();
  const { data } = useSuspenseQuery({
    queryKey: ["hello"],
    queryFn: async () => {
      return http.get("/hello");
    },
  });
  return (
    <FormProvider {...form}>
      <form
        onSubmit={form.handleSubmit((data) => {
          mutate(data, {
            onSuccess: () => {
              //
            },
          });
        })}
      >
        <FormField
          control={form.control}
          name={"email"}
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name={"password"}
          render={({ field }) => (
            <FormItem>
              <FormLabel>Password</FormLabel>
              <FormControl>
                <input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
      </form>
    </FormProvider>
  );
};

```

### @repo/ui

대부분의 컴포넌트 라이브러리들에서 기본적으로 사용되는 UI 컴포넌트를 제공합니다.

이 컴포넌트를 활용하여 자주 사용하게 되는 UI를 구성할 수 있습니다.

자세한 스펙과 사용가능한 컴포넌트는 ./packages에서 확인하세요


```tsx
import { Box } from "@repo/ui/Box";
import { JustifyEnd } from "@repo/ui/JustifyEnd";
import { List } from "@repo/ui/List";
import { Row } from "@repo/ui/Row";
import { Stack } from "@repo/ui/Stack";

function App() {
  return (
    <>
      <Stack>
        <List
          fallback={<div>리스트가 빈경우 보여주는 폴백입니다.</div>}
          with={<div>리스트의 아이템 사이에 세퍼레이터가 필요한 경우 사용합니다.</div>}
        >
          <Row
            left={<div>컴포넌트 왼쪾에 뭔가를 배치해야하는 경우 사용합니다.</div>}
            right={<div>오른쪽 배치</div>}
          ></Row>
          <Row className=" gap-x-4" left={<div className=" h-[16px] w-[16px] bg-red-500" />}>
            <Box>hello</Box>
          </Row>
        </List>
        <JustifyEnd>오른쪽에 물체를 배치해야하는 경우 사용합니다.</JustifyEnd>
      </Stack>
    </>
  );
}

export default App;

```

### @repo/env

필요로하는 환경변수가 빠져있는 경우 에러를 발생시켜주는 패키지입니다.

로컬 개발을 하고있는 경우 .env.local을 작성해주세요

작성 예시

.env.local

```
VITE_API_URL=http://localhost:3000
```

### @repo/mocks

백엔드 api가 나오지 않은 경우 mocks를 이용하여 백엔드 응답을 모킹합니다. msw를 기반으로 동작합니다.
