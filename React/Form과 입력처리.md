# Form과 입력 처리

React에서 폼을 다루는 방법은 기존 HTML과 다릅니다. React는 **Contolled Component(제어 컴포넌트)** 와 **Uncontrolled Component(비제어 컴포넌트)** 두 가지 방식으로 폼 요소를 처리할 수 있습니다.

<br />

## **Controlled Component (제어 컴포넌트)**

제어 컴포넌트는 **React의 state가 "신뢰 가능한 단일 출처(single source of truth)"** 가 되는 방식입니다. 폼 요소의 값이 항상 React state와 동기화되며, 모든 변경사항이 state를 통해 관리됩니다.

### 기본 원리

1. 폼 요소의 값을 state로 관리

2. 사용자 입력 시 onChange 이벤트로 state 업데이트

3. state가 변경되면 컴포넌트가 리렌더링되어 입력 요소의 값도 업데이트

```jsx
function ControlledInput() {
  const [value, setValue] = useState("");

  const handleChange = (e) => {
    setValue(e.target.value);
  };

  return (
    <div>
      <input
        type="text"
        value={value} // state와 연결
        onChange={handleChange} // state 업데이트
      />
      <p>입력한 값: {value}</p>
    </div>
  );
}
```

### 다양한 입력 타입 처리

**1. Text Input과 Textarea**

```jsx
function TextInputExample() {
  const [formData, setFormData] = useState({
    username: "",
    bio: "",
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: value,
    }));
  };

  return (
    <form>
      <input
        type="text"
        name="username"
        value={formData.username}
        onChange={handleChange}
        placeholder="사용자명"
      />
      <textarea
        name="bio"
        value={formData.bio}
        onChange={handleChange}
        placeholder="자기소개"
        rows={4}
      />
    </form>
  );
}
```

**2. Checkbox**

```jsx
function CheckboxExample() {
  const [isChecked, setIsChecked] = useState(false);
  const [hobbies, setHobbies] = useState({
    reading: false,
    gaming: false,
    coding: true,
  });

  // 단일 체크박스
  const handleSingleCheck = (e) => {
    setIsChecked(e.target.checked);
  };

  // 다중 체크박스
  const handleHobbyChange = (e) => {
    const { name, checked } = e.target;
    setHobbies((prev) => ({
      ...prev,
      [name]: checked,
    }));
  };

  return (
    <div>
      {/* 단일 체크박스 */}
      <label>
        <input type="checkbox" checked={isChecked} onChange={handleSingleCheck} />
      </label>

      {/* 다중 체크박스 */}
      <fieldest>
        <legend>취미:</legend>
        <label>
          <input
            type="checkbox"
            name="reading"
            checked={hobbies.reading}
            onChange={handleHobbyChange}
          />
          독서
        </label>
        <label>
          <input
            type="checkbox"
            name="coding"
            checked={hobbies.coding}
            onChange={handleHobbyChange}
          />
          코딩
        </label>
      </fieldest>
    </div>
  );
}
```

**3. Radio Button**

```jsx
function RadioExample() {
  const [selectedOption, setSelectedOption] = useState("option1");

  const handleRadioChange = (e) => {
    setSelectedOption(e.target.value);
  };

  return (
    <fieldset>
      <legend>배송 옵션을 선택하세요:</legend>
      <label>
        <input
          type="radio"
          value="standard"
          checked={selectedOption === "standard"}
          onChange={handleRadioChange}
        />
        일반 배송(무료)
      </label>
      <label>
        <input
          type="radio"
          value="express"
          checked={selectedOption === "express"}
          onChange={handleRadioChange}
        />
        특급 배송(3,000원)
      </label>
      <label>
        <input
          type="radio"
          value="overnight"
          checked={selectedOption === "overnight"}
          onChange={handleRadioChange}
        />
        익일 배송(5,000원)
      </label>
      <p>선택한 옵션: {selectedOption}</p>
    </fieldset>
  );
}
```

**4. Select (Dropdown)**

```jsx
function SelectExample() {
  const [country, setCountry] = useState("kr");
  const [languages, setLanguages] = useState([]); // 다중 선택

  // 단일 선택
  const handleCountryChange = (e) => {
    setCountry(e.target.value);
  };

  // 다중 선택
  const handleLanguageChange = (e) => {
    const options = e.target.options;
    const selected = [];
    for (let i = 0; i < options.length; i++) {
      if (options[i].selected) {
        selected.push(options[i].value);
      }
    }
    setLanguages(selected);
  };

  return (
    <div>
      {/* 단일 선택 */}
      <select value={country} onChange={handleCountryChange}>
        <option value="">국가를 선택하세요</option>
        <option value="kr">한국</option>
        <option value="us">미국</option>
        <option value="jp">일본</option>
      </select>

      {/* 다중 선택 */}
      <select multiple value={languages} onChange={handleLanguageChange}>
        <option value="javascript">JavaScript</option>
        <option value="python">Python</option>
        <option value="java">Java</option>
        <option value="go">Go</option>
      </select>
    </div>
  );
}
```

<br />

## Uncontrolled Component (비제어 컴포넌트)

비제어 컴포넌트는 **DOM 자체가 폼 데이터를 관리**하는 방식입니다. React는 `ref`를 통해 필요할 때만 DOM에서 값을 가져옵니다.

### useRef를 사용한 비제어 컴포넌트

```jsx
function UncontrolledForm() {
  const inputRef = useRef(null);
  const fileRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("입력값", inputRef.current.value);
    console.log("파일", fileRef.current.files[0]);
  };

  return (
    <form onSubnit={handleSubmit}>
      <input
        ref={inputRef}
        type="text"
        defaultValue="초기값" // value 대신 defaultValue 사용
      />

      <input
        ref={fileRef}
        type="file" // 파일 입력은 비제어 컴포넌트로만 가능
      />
      <button type="submit">제출</button>
    </form>
  );
}
```

### 파일 입력 처리

파일 입력은 읽기 전용이므로 항상 비제어 컴포넌트입니다.

```jsx
function FileUpload() {
  const [selectedFile, setSelectedFile] = useState(null);
  const [preview, setPreview] = useState(null);

  const handleFileChange = (e) => {
    const file = e.target.files[0];
    if (file) {
      setSelectedFile(file);

      // 이미지 미리보가
      if (file.type.startWith("image/")) {
        const reader = new FileReader();
        reader.onloadend = () => {
          setPreview(reader.result);
        };
        reader.readAsDataURL(file);
      }
    }
  };

  return (
    <div>
      <input
        type="file"
        onChange={handleFileChange}
        accept="image/*" // 이미지 파일만 허용
      />

      {selectedFile && (
        <div>
          <p>파일명: {selectedFile.name}</p>
          <p>크기: {(selectedFile.size / 1024).toFixed(2)} KB</p>
          <p>타입: {selectedFile.type}</p>
        </div>
      )}

      {preview && <img src={preview} alt="미리보가" style={{ maxWidth: "200px" }} />}
    </div>
  );
}
```

<br />

## 폼 유효성 검사 (Form Validation)

### 실시간 유효성 검사

```jsx
function FormWithValidation() {
  const [formData, setFormData] = useState({
    email: "",
    password: "",
    confirmPassword: "",
  });

  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  // 유효성 검사 함수
  const validate = (name, value) => {
    const newErroes = { ...errors };

    switch (name) {
      case "email":
        if (!value) {
          newErrors.email = "이메일을 입력하세요";
        } else if (!/\S+@\.\S+/.test(value)) {
          newErrors.email = "올바른 이메일 형식이 아닙니다";
        } else {
          delete newErroes.email;
        }
        break;

      case "password":
        if (!value) {
          newErrors.password = "비밀번호를 입력하세요";
        } else if (value.length < 8) {
          newErrors.password = "비밀번호는 8자 이상이어야 합니다";
        } else {
          delete newErrors.password;
        }

        // 비밀번호 확인도 재검증
        if (formData.confirmPassword && value !== formData.confirmPassword) {
          newErrors.confirmPassword = "비밀번호가 일치하지 않습니다";
        } else if (formData.confirmPassword) {
          delete newErrors.confirmPassword;
        }
        break;

      case "confirmPassword":
        if (!value) {
          newErrors.confirmPassword = "비밀번호 확인을 입력하세요";
        } else if (value.length < 8) {
          newErrors.password = "비밀번호가 일치하지 합니다";
        } else {
          delete newErrors.password;
        }
        break;

     default:
        break;
    }

    setErrors(newErrors);
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
        ...prev,
        [name]: value
    }));

    // 입력 중 실시간 검증 {touched된 필드만}
    if(touched[name] {
        validate(name, value);
    })
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({
        ...prev,
        [name]: true
    }));
    validate(name, formData[name]);
  };

  const handleSubmit = (e) => {
    e.preventDefault();

    // 모든 필드 검증
    Object.keys(formData).forEach(key => {
        validate(key, formData[key]);
    });

    // 에러가 없으면 제출
    if (Object.keys(errors).length === 0) {
        console.log('폼 제출:', formData);
        // API 호출 등 처리
    } else {
        console.log('유효성 검사 실패');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="이메일"
          className={errors.email ? 'error' : ''}
        />
        {errors.email && touched.email && (
          <span className="error-message">{errors.email}</span>
        )}
      </div>

      <div>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="비밀번호"
          className={errors.password ? 'error' : ''}
        />
        {errors.password && touched.password && (
          <span className="error-message">{errors.password}</span>
        )}
      </div>

      <div>
        <input
          type="password"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="비밀번호 확인"
          className={errors.confirmPassword ? 'error' : ''}
        />
        {errors.confirmPassword && touched.confirmPassword && (
          <span className="error-message">{errors.confirmPassword}</span>
        )}
      </div>

      <button
        type="submit"
        disabled={Object.keys(errors).length > 0}
      >
        회원가입
      </button>
    </form>
  );
}
```

<br />

## Custom Hook으로 폼 로직 재사용

반복되는 폼 로직을 Custom Hook으로 추출하면 재사용성이 높아집니다.

```jsx
// useForm.js
function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setValues((prev) => ({
      ...prev,
      [name]: type === "checkbox" ? checked : value,
    }));

    // 실시간 검증
    if (validate && touched[name]) {
      const validationErrors = validate({ ...values, [name]: value });
      setErrors(validationErrors);
    }
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched((prev) => ({
      ...prev,
      [name]: true,
    }));

    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
    }
  };

  const handleSubmit = (callback) => (e) => {
    e.prevntDefault();

    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      if (Object.keys(validationErrors).length === 0) {
        setIsSubmitting(true);
        callback(values);
        setIsSubmitting(false);
      } else {
        callback(values);
      }
    }
  };

  const resetForm = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  };

  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
  };
}
```

```jsx
// LoginForm.js
function LoginForm() {
  const { values, errors, touched, handleChange, handleBlur, handleSubmit } = useForm(
    { email: "", password: "" },
    validate
  );

  const validate = (values) => {
    const errors = {};
    if (!values.email) {
      errors.email = "이메일을 입력하세요";
    }
    if (!values.password) {
      errors.password = "비밀번호를 입력하세요";
    }
    return errors;
  };

  const onSubmit = (data) => {
    console.log("로그인 데이터", data);
    // API 호출
  };
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        name="email"
        type="email"
        value={values.email}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {errors.email && touched.email && <span>{errors.email}</span>}

      <input
        name="password"
        type="password"
        value={values.password}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {errors.password && touched.password && <span>{errors.password}</span>}

      <button type="submit">로그인</button>
    </form>
  );
}
```

<br />

## Controlled vs Uncontroolled 비교

| **특징**           | **제어 컴포넌트**  | **비제어 컴포넌트**  |
| ------------------ | ------------------ | -------------------- |
| 데이터관리         | React state로 관리 | DOM이 직접 관리      |
| 실시간 유효성 검사 | 쉬움               | 어려움               |
| 조건부 비활성화    | 쉬움               | 어려움               |
| 동적 입력          | 쉬움               | 어려움               |
| 파일 입력          | 불가능             | 가능                 |
| 성능               | 리렌더링 발생      | 리렌더링 최소화      |
| 사용 시점          | 대부분의 경우      | 간단한 폼, 파일 입력 |

<br />

## 실전 팁

1. **대부분의 경우 제어 컴포넌트를 사용합니다.** 데이터 흐름이 명확하고 디버깅이 쉽습니다.

2. **파일 입력은 항상 비제어 컴포넌트 입니다.**

3. **폼이 복잡해지면 폼 라이브러리 사용을 고려합니다.** (React Hook, Form, Formik 등)

4. **디바운싱을 활용하여 성능을 최적화합니다.**

```jsx
const [searchTerm, setSearchTerm] = useState("");
const [debouncedTerm, setDebouncedTerm] = useState("");

useEffect(() => {
  const timer = setTimeout(() => {
    setDebouncedTerm(searchTerm);
  }, 500);

  return () => clearTimeout(timer);
}, [searchTerm]);
```

5. **입력 포맷팅이 필요한 경우 조심합니다.**(전화번호, 카드번호 등)

<br />

# 참고 자료

> [React 공식 문서 - 폼](https://ko.react.dev/reference/react-dom/components/form)

> [React 공식 문서 - Controlled vs Uncontrolled Components](https://ko.react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components)

> [MDN - HTML 폼 가이드](https://developer.mozilla.org/ko/docs/Learn/Forms)
