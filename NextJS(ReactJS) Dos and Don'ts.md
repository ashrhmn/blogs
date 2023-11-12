## Always and always use `key` prop in lists rendered using `map`

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

## Use ESLint rule to enforce best practices suggested by NextJS

```json
{
  "extends": "next/core-web-vitals"
}
```

#### You can always override default rules using the rules property

For example, if you want to disable the `no-img-element` rule, you can do so by adding the following to your `.eslintrc.json` file. Follow ESLint documentation for more details.

```json
{
  "extends": "next/core-web-vitals",
  "rules": {
    "@next/next/no-img-element": "off"
  }
}
```

## Make sure you do not send request to the backend with `undefined` param

Often, there's a request like this:

`https://example.com/api/posts-by-user-id/undefined`

or

`https://example.com/api/posts-by-user-id/undefined/categories`

Why make the extra request, make round trip to the backend and hung up backend with `Internal Server Error` if the param error is not handled well? You get extra rejected requests as well when backend throws the error as `undefined is not a valid number/UUID`. Always make sure you have a valid `id` before sending the request.

If you are using `react-query`, it is also really easy to handle as well,

```js
const router = useRouter();
const blogId = router.query.id;

const { data: blogData } = useQuery({
  queryKey: ["blog", blogId],
  queryFn: () => axios.get(`/api/blogs/${blogId}`),
  enabled: !!blogId, // when blogId is undefined, the query will simply not be sent
});
```

## DRY (Don't Repeat Yourself)

Often, we have code like this,

```jsx
<Image src={config.API_BASE_URL + "public/uploads" + user.image} />
```

And you have the same damn thing in 40 different files and 5 times in each file. The prefix `public/uploads` can get changed. You will have to change it in 40 different files. Instead just make an utility function and use it everywhere.

```js
// Pipe to a URL sanitizer if possible
const constructStaticFileUrl = (...paths:string[])=> config.API_BASE_URL+"public/uploads"+paths.join("");

<Image src={constructStaticFileUrl(user.image)} />
<Image src={constructStaticFileUrl(user.id,"images","cover.png")} />
```

## The middle ground for DRY Pattern

DRY pattern is a good pattern to follow. But sometimes, it is not always the best. For example, you have a component that is used in 2 different pages. You want to make sure that the component is not duplicated. So you create a generic component and use it in both pages. But what if you want to change something in the component in one page but not the other? You will add conditional logic to the component to make sure it renders differently in different pages. Now, you are getting more and more logic coming for the component and you keep adding more and more logic to it. It is getting harder to maintain. What to do?-

- If the component is duplicated just 2 times, it is okay to duplicate it. It is not the end of the world. It is better to have 2 components that are easy to maintain than 1 component that is hard to maintain.

- Decide a middle ground (eg. 3 times) and if the component is duplicated more than 3 times, then create a separate generic component.

- If your generic component is getting too complex with tons of logic, break that component into smaller components as well and use logic only to the smaller components where necessary.

- If it is still hard to maintain the logics for each usages, create a separate hook for the logic or create different hooks for different logics and use inside the component where needed.

## Stop over using `console.log`

If you are using `console.log` to debug your components, that's fine. But make sure to clean it up when you are done. When some other developer is working on a separate thing and he is being blown away with `console.log`s from your component, it is not a good experience for him. It is also not a good experience for you when you are debugging some other component and you are getting confused with `console.log`s from other component and you do not know where these logs are coming from. If one of your component is throwing errors with the React tree, and excessive `console.log`s are pushing them up and up, you will not be able to see easily that you have a error up there. So, make sure to clean up your `console.log`s when you are done.

## Use errors and warnings as todos

Do not leave the warnings and errors hanging around throughout the application. If you know better than the compiler or the interpreter (Yes, sometimes we do) that it should not raise the warning or the error, just make it silent. ESLint does support silencing errors and warnings using inline comments. Refer to the ESLint documentation for more details.
