---
title: Non Null Assertion vs as로 타입 단언
date: '2022-12-18'
lastmod: '2022-12-18'
tags: ['typescript']
draft: false
summary: ''
---

코드스테이츠 메인 프로젝트를 진행하면서, Editor에 이미지를 넣는 기능을 구현을 담당했었다. 이 기능을 구현하면서 타입을 지정하는 과정에서 `Non Null Assertion`과 `as`로 타입 단언을 하는 것에 대한 차이가 궁금해졌다.

```tsx
const quillRef = useRef<ReactQuill>(null)
const imageHandler = useCallback(async () => {
  const input = document.createElement('input')
  input.setAttribute('type', 'file')
  input.setAttribute('accept', 'image/*')
  document.body.appendChild(input)

  input.click()
  input.onchange = async () => {
    if (input.files) {
      const file = input.files[0]

      const { preSignedUrl, fileId } = await getFileUrl()
      await uploadImg(preSignedUrl, file)
      const imageUrl = preSignedUrl.split('png')[0] + 'png'

      fileIdList.push({ fileId })
      const newFiledIdList = fileIdList
      setFileIdList(newFiledIdList)

      const range = quillRef.current?.getEditorSelection()
      if (quillRef.current && range) {
        if (typeof range.index === 'number')
          setTimeout(() => {
            const index = range.index
            quillRef.current?.getEditor().insertEmbed(index, 'image', imageUrl)
            quillRef.current?.getEditor().setSelection({ index: range.index + 1, length: 0 })
            const myInput = document.body.querySelector(':scope > input') as HTMLInputElement
            myInput.remove()
          }, 500)
      } else {
        console.error('Error: range is null.')
      }
    }
  }
}, [])
```

- 2023.02.20 수정
  먼저 `quillRef`에 대해 타입을 지정해줬고 타입 가드를 이용해서 해결하였다.

아무튼 이 두가지 방식의 타입 단언을 비교해 보려고 한다.

### Non-null assertion

```ts
interface Foo {
  bar?: string
}

const foo: Foo = getFoo()
const includesBaz: boolean = foo.bar!.includes('baz')
```

이 예제에서는 `foo.bar`가 `null`이나 `undefined`가 아니라고 확신할 때 쓴다.

### as를 통한 타입 단언

타입 단언을 되도록이면 피해야 되는 이유는 첫 번째로 타입 체크를 할 수가 없기 때문이다.

```ts
interface Person {
  name: string
}

const alice: Person = { name: 'Alice' } // 타입은 Person
const bob = { name: 'Bob' } as Person // 타입은 Person

const alice: Person = {}
//    ~~~~~ 'Person' 유형에 필요한 'name' 속성이 '{}' 유형에 없습니다.
const bob = {} as Person // 오류 없음
```

두 번째 이유로 타입 선언 방식을 사용했을 때는 속성 추가 시에 오류가 나지만 타입 단언을 사용했을 때는 아무런 오류가 나타나지 않기 때문이다.

```ts
const alice: Person = {
  name: 'Alice',
  occupation: 'TypeScript Developer',
  // ~~~~~~~ 개체 리터럴은 알려진 속성만 지정할 수 있음, Person 형식에 occupation이 없습니다.
}

const bob = {
  name: 'Bob',
  occupation: 'Javascript Developer',
} as Person // 오류 없음
```

이제 슬슬 이펙티브 타입스크립트도 읽어야겠다...

### Reference

<a href='https://github.com/typescript-eslint/typescript-eslint/blob/main/packages/eslint-plugin/docs/rules/no-non-null-assertion.md'>no-non-null-assertion</a>

이펙티브 타입스크립트
