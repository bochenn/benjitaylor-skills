# Avatar

Change the avatar used in ConnectKit to more closely match the look and feel of your app.

By default ConnectKit has an avatar component that generates a random gradient based on the user's wallet address in case their ENS image is not set. You can customize this by providing your own avatar component.

## Example

First import `Types`, then create a custom avatar component:

```tsx
// MyCustomAvatar.tsx
import { Types } from "connectkit";

const MyCustomAvatar = ({ address, ensImage, ensName, size, radius }: Types.CustomAvatarProps) => {
  return (
    <div
      style={{
        overflow: "hidden",
        borderRadius: radius,
        height: size,
        width: size,
        background: generateColorFromAddress(address), // your function here
      }}
    >
      {ensImage && <img src={ensImage} alt={ensName ?? address} width="100%" height="100%" />}
    </div>
  );
};

export default MyCustomAvatar;
```

Then apply your avatar component to the `<ConnectKitProvider>` via the `customAvatar` option:

```tsx
// _app.tsx
import MyCustomAvatar from "./MyCustomAvatar";

<ConnectKitProvider
  options={{
    customAvatar: MyCustomAvatar,
  }}
>
  {/* Your App */}
</ConnectKitProvider>;
```

That's it—you will now have replaced all usage of the `<Avatar />` component used within ConnectKit.
