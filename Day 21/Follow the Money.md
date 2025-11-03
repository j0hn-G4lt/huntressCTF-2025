- https://evergatetltle.netlify.app/
- Send a fake transfer

```
Details Submitted
Thanks for giving me your bank! Your friend, aHR0cHM6Ly9uMHRydXN0eC1ibG9nLm5ldGxpZnkuYXBwLw== Retrieval ID: 471082
```

- b64 decode

```sh
echo 'aHR0cHM6Ly9uMHRydXN0eC1ibG9nLm5ldGxpZnkuYXBwLw==' | base64 -d
https://n0trustx-blog.netlify.app/
```

- Go to their Github
- Hacker username = `N0TrustX`
- Inspect https://github.com/N0TrustX/Spectre/blob/main/spectre.html

```html
<!-- The Base64 encoded object is stored here, hidden from view -->
<div id="encodedPayload" class="hidden">
  ZmxhZ3trbDF6a2xqaTJkeWNxZWRqNmVmNnltbHJzZjE4MGQwZn0=
</div>
<button
  id="closePayloadBtn"
  class="mt-8 action-button font-bold py-2 px-6 rounded-lg"
>
  Close
</button>
```

- b64 decodes to `flag{kl1zklji2dycqedj6ef6ymlrsf180d0f}`
