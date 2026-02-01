---
title: About
linkTitle: About
menu: {main: {weight: 10}}
---

{{% blocks/cover title="About" height="auto" %}}

{{% /blocks/cover %}}

{{% blocks/section color="primary" %}}

Hello,

If you have ideas or feedback about RsyncUI, feel free to reach me at thomeven@gmail.com.

I started RsyncOSX in August 2016 as a way to learn Swift. After eight years of releases, it was archived in August 2024. RsyncUI, built with SwiftUI, began in late 2020 and is still actively developed.

I hold a master's degree in computing science, earned in the early 1990s when Linux and the public Internet were emerging. The Web itself was invented just a few years earlier, in 1989 at CERN. I'm a solo developer focused on keeping RsyncUI stable and usable. I'm not a professional UI designer, so parts of the interface may feel dense.

Maintaining RsyncUI is a hobby. If you find it helpful, a GitHub star is appreciated—it keeps me motivated. I retired in May 2022 at age 62. Outside coding, I am a [bird photographer](https://birdsofprey.netlify.app) who enjoys time in the Norwegian mountains. Grandchildren, photography, RsyncUI, and cross-country skiing keep me busy.

{.text-center}

{{% /blocks/section %}}

{{% blocks/section color="white" %}}

Although I am an educated IT professional, most of my career was in IT management, not hands-on development. My coding experience comes from personal projects like RsyncUI and RsyncOSX. Search, documentation, and learning from other developers' examples have been invaluable.

I keep learning with every Swift, SwiftUI, and Xcode release. Major changes and updates are logged in the [changelog](/blog/). I rely on tools such as [SwiftLint](https://github.com/realm/SwiftLint), [SwiftFormat](https://github.com/nicklockwood/SwiftFormat), and [periphery](https://github.com/peripheryapp/periphery) to improve code quality.

**Why Not the App Store?**

One crucial requirement for macOS applications on the Apple App Store, as outlined by Apple, is: *"To distribute a macOS app through the Mac App Store, you must enable the App Sandbox capability."* The App Sandbox imposes certain restrictions on the functionality of an application within its environment. While it is essential for enabling passwordless login via SSH to remote servers, it also causes some limitations with these features when enabled.

{.text-center}

{{% /blocks/section %}}

{{% blocks/section color="primary" %}}

**How are these pages constructed?**

Hugo, a static site generator, serves as the web framework responsible for constructing these pages. The source code for this website is hosted on GitHub. Netlify, the web server, automatically detects changes made to the main branch and promptly rebuilds the server. The Hugo theme utilized is [docsy](https://github.com/google/docsy), another open-source project hosted on GitHub. Upon making modifications or additions to pages, I commit these changes to GitHub, and Netlify promptly constructs the new server within seconds.

{.text-center}

{{% /blocks/section %}}