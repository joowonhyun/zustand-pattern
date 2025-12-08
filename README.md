# Zustand Store Patterns

## Basic Pattern

```TypeScript
import { create } from "zustand";

interface UserState {
  name: string;
  age: string;
  increaseAge: (by: number) => void;
}

const useUserStore = create<UserState>()((set) => ({
  name: "홍길동",
  age: "20",
  increaseAge: (by) => set((state) => ({ age: state.age + by })),
}));
```

## Slice Pattern

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

// 합쳐질 최종타입(slice 모음), [['zustand/devtools', never]](미들웨어 타입), [](미들웨어 타입), 현재 슬라이스 타입
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

## Usage
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

## Middleware

```TypeScript
import { create } from "zustand";
import {
  combine,
  subscribeWithSelector,
  persist,
  createJSONStorage,
  devtools,
} from "zustand/middleware";
import { immer } from "zustand/middleware/immer";

export const useCountStore = create(
// 순서 중요
  devtools(
    persist(
      subscribeWithSelector(
        immer(
          combine({ count: 0 }, (set, get) => ({
            actions: {
              increaseOne: () => {
                set((state) => {
                  state.count += 1;
                });
              },
              decreaseOne: () => {
                set((state) => {
                  if (state.count > 0) {
                    state.count -= 1;
                  }
                });
              },
            },
          })),
        ),
      ),
      {
        name: "countStore",
        partialize: (store) => ({
          count: store.count,
        }),
        storage: createJSONStorage(() => sessionStorage),
      },
    ),
    {
      name: "countStore",
    },
  ),
);

useCountStore.subscribe(
  (store) => store.count,
  (count, prevCount) => {
    //Listener
    console.log(count, prevCount);
    const store = useCountStore.getState();
    useCountStore.setState((store) => ({}));
  },
);

export const useCount = () => {
  const count = useCountStore((store) => store.count);
  return count;
};

export const useIncreaseCount = () => {
  const increase = useCountStore((store) => store.actions.increaseOne);
  return increase;
};

export const useDecreaseCount = () => {
  const decrease = useCountStore((store) => store.actions.decreaseOne);
  return decrease;
};

```
