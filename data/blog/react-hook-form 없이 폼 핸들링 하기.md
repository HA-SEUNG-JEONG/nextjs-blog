---
title: react-hook-form 없이 form 핸들링 하기
date: '2023-07-14'
lastmod: '2023-07-14'
tags: ['글또', 'react', 'react-hook-form']
draft: false
summary: 'react-hook-form 없이 폼 핸들링 작업을 진행했습니다.'
---

### 시작한 이유

react-hook-form을 작년 10월에 처음 접했는데, 사용하다보니 너무 라이브러리에만 의존하고 있다는 생각이 들었고 hook으로 만들어서 관리해보기로 했는데 검색을 하다 보니 좋은 코드가 있어서 참고하면서 시작했다.

#### handleChange

`handleChange` 함수에서는 사용자가 입력한 값과 이 값을 특정 key에 매칭하도록 했다.

```tsx
const handleChange = (key: keyof T) => (event: React.ChangeEvent<HTMLInputElement>) => {
  setData((prevData) => ({ ...prevData, [key]: event.target.value }))
}
```

#### handleSubmit

`handleSubmit` 함수는 좀 길어서 나눠서 설명하려고 한다.

우선 react-hook-form을 사용할 때는 `event.preventDefault()` 를 직접 작성할 필요는 없지만 커스텀 훅을 만들때는 필요한 과정이다.

라이브러리에 정말 `event.preventDefault()` 가 진짜 내장되어 있는지 확인하기 위해 해당 repository에 있는 코드를 좀 까봤다.

```tsx
const handleSubmit: UseFormHandleSubmit<TFieldValues> = (onValid, onInvalid) => async (e) => {
  if (e) {
    e.preventDefault && e.preventDefault() // 여기서 preventDefault 실행
    e.persist && e.persist()
  }
  // 나머지 코드 생략
}
```

<strong><p align='center'><span>https://github.com/react-hook-form/react-hook-form/blob/master/src/logic/createFormControl.ts</span></p></strong>

handleSubmit이 실행될 때 preventDefault가 동작하도록 설계된 것이 맞다.

```tsx
const setValidationError = (key: string, message: string) =>
  setErrors((prevErrors) => ({ ...prevErrors, [key]: message }))

if (validation?.required?.value && !value) {
  setValidationError(key, validation?.required?.message)
  return
}

if (pattern?.value && !pattern?.value?.test(value as string)) {
  setValidationError(key, pattern?.message)
  return
}

if (custom?.isValid && (!isValidString(value) || !custom?.isValid(value as string))) {
  setValidationError(key, custom?.message)
  return
}
```

`setValidationError` 라는 함수를 이용해서 특정 값이 정규식 패턴에 맞지 않거나, 다른 조건(length 등)에 만족하지 못하면 에러를 내도록 했다.

```tsx
setErrors({}) // 에러 없애기

if (!isSubmitting) {
  alert('제출되었습니다!')
  return
}
```

이후 모든 조건을 다 만족하고 submit 버튼을 눌렀을 때 `setErrors` 함수를 통해 에러를 없애주도록 했다.

하지만 폼 제출을 하면서 alert 창이 여러번 뜨는 현상이 일어났는데, handleSubmit이 form 전체에 걸쳐져 있어서 그런거 같다고 생각이 들어 `alert('제출되었습니다!')` 가 한번 뜨면 return 처리해서 바로 끝내도록 했다.

이게 적절한 방법인지는 더 고민해봐야 할 것 같다..

### useForm 훅 적용

`Registration.tsx` 파일에서 `useForm` hook을 적용해보았다.

react-hook-form에 있는 `register` 함수와 네이밍은 똑같이 가져갔고, `name`, `value`, `onChange`를 key로 받아서 사용했다.

추가적으로 input 필드에 값이 아무것도 없는 상태에서는 submit 버튼을 disable 처리하도록 하고 싶었다.

```tsx
const isFormEmpty = Object.values(user).every((value) => value === '')
isFormEmpty ? setIsValid(false) : setIsValid(true)
```

단순히 이렇게만 작성하니 제대로 동작하지 않았는데,

`useEffect`를 사용해서 user 값 변화에 따라 `isFormEmpty` 함수를 통해 value가 빈 값인지 판단하도록 했다.

전체 코드(interface 제외)

```tsx
const Registration = () => {
  const [isValid, setIsValid] = useState(false)
  const {
    handleSubmit,
    handleChange,
    data: user,
    errors,
  } = useForm<User>({
    validations: validationRules,
  })

  const register = (key: keyof User) => ({
    name: key,
    value: user[key] || '',
    onChange: handleChange(key),
  })

  // 유저 정보 변화에 따라 isFormEmpty 함수 실행
  useEffect(() => {
    const isFormEmpty = Object.values(user).every((value) => value === '')
    isFormEmpty ? setIsValid(false) : setIsValid(true)
  }, [user])

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="Name" type="text" {...register('name')} />
      {errors.name && <p>{errors.name}</p>}
      <input placeholder="age" type="number" {...register('age')} />
      {errors.age && <p>{errors.age}</p>}
      <input placeholder="Email" type="email" {...register('email')} />
      {errors.email && <p>{errors.email}</p>}
      <input placeholder="Password" type="password" {...register('password')} />
      {errors.password && <p>{errors.password}</p>}
      <button type="submit" disabled={!isValid}>
        Submit
      </button>
    </form>
  )
}
```

### 마무리

타입스크립트로 hook을 만들면서 이제야 좀 제네릭이라는 것에 대해 익숙해진 느낌이 든다.

추가적으로 CSS적인 부분도 작성해야 하고, 로직에 이상이 없는지는 더 살펴봐야 한다.

조금만 더 손을 보면 큰 프로젝트가 아닌 이상은 웬만하면 react-hook-form 쓸 일은 별로 없을 것 같다.

https://github.com/HA-SEUNG-JEONG/form-handling-without-library 에서 전체 코드를 확인하실 수 있습니다.

### Reference

https://github.com/fgerschau/react-custom-form-validation-example
