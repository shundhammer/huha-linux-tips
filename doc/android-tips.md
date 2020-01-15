# HuHa's Android Tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License

## Adblock via Adguard-DNS (No Root)

On the Android device:

- Settings app
- Tab "Wireless and Network"
- "Private DNS"
- Switch from "Auto" to "Configure private DNS"
- In the hostname field, enter `dns.adguard.com`
- Do **not** hit the `Return` key!

The Adguard DNS is now used for all apps. But the Chrome browser may still need a special flag:

- Chrome browser
- Enter URL `chrome://flags`
- In the "Search flags" input field, enter `dns`
- Select "Async DNS resolver"
- Set to "Disabled"
