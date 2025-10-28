# zustand 보일러 플레이트

## Basic Pattern Version

```TypeScript
import { create } from "zustand";

interface UserState {
  name: string;
  age: string;
  increaseAge: (by: number) => void;
}

const useUserStore = create<UserState>((set) => ({
  name: "홍길동",
  age: "20",
  increaseAge: (by) => set((state) => ({ age: state.age + by })),
}));
```

## Slice Pattern Version

```TypeScript
import { create } from "zustand";

interface UserSlice {
  name: string;
  age: number;
  increaseAge: (by: number) => void;
}

interface SettingSlice {
  theme: "light" | "dark";
  toggleTheme: () => void;
}

// 합쳐질 최종타입 , [](미들웨어 타입), [](미들웨어 타입), 현재 슬라이스 타입
const creatUserSlice: StateCreator<
  UserSlice & SettingSlice,
  [],
  [],
  UserSlice
> = (set) => ({
  name: "홍길동",
  age: 30,
  increaseAge: () =>
    set((state) => ({
      age: state.age + 1,
    })),
});

// 합쳐질 최종타입 , [](미들웨어 타입), [](미들웨어 타입), 현재 슬라이스 타입
const createSettingSlice: StateCreator<
  UserSlice & SettingSlice,
  [],
  [],
  SettingSlice
> = (set) => ({
  theme: "light",
  toggleTheme: () =>
    set((state) => ({ theme: state.theme === "light" ? "dark" : "light" })),
});

const useUserStore = create<UserSlice & SettingSlice>()((...a) => ({
  ...creatUserSlice(...a),
  ...createSettingSlice(...a),
}));

export default useUserStore;


```

## 컴포넌트 활용
```TypeScript
import useUserStore from "./store/useUserStore";
import { useShallow } from "zustand/react/shallow";

export default function App() {
  const { name, age, increaseAge, theme, toggleTheme } = useUserStore(
    useShallow((state) => ({
      name: state.name,
      age: state.age,
      increaseAge: state.increaseAge,
      theme: state.theme,
      toggleTheme: state.toggleTheme,
    }))
  );

  return (
    <div>
      <h1>Zustand with TypeScript</h1>
      <p>name: {name}</p>
      <p>age: {age}</p>
      <p>theme: {theme}</p>
      <button onClick={() => increaseAge(20)}>나이 증가</button>
      <button onClick={toggleTheme}>테마 변경</button>
    </div>
  );
}
```
