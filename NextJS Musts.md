Must follow rules for a NextJS Application

## Use `key` prop in lists rendered using `map`

When rendering lists using `map`, it is important to provide a `key` prop to the topmost element in the list. This helps React identify which items have changed, are added, or are removed. This is important for performance reasons as well as to avoid bugs where React mixes up the state of items in the list.

```jsx
const list = [1, 2, 3, 4, 5];

const List = () => {
  return (
    <ul>
      {list.map((item) => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
};
```
