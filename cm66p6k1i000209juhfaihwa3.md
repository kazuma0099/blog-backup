---
title: "[Error] Intellij 2022.1.4 Eslint"
seoTitle: "Error"
seoDescription: "Eslint intellij error"
datePublished: Tue Jan 21 2025 16:36:37 GMT+0000 (Coordinated Universal Time)
cuid: cm66p6k1i000209juhfaihwa3
slug: error-intellij-202214-eslint
tags: error

---

```bash
TypeError: this.libOptions.parse is not a function

TypeError: this.libOptions.parse is not a function
    at ESLint8Plugin.<anonymous> (/Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:139:64)
    at step (/Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:44:23)
    at Object.next (/Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:25:53)
    at /Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:19:71
    at new Promise (<anonymous>)
    at __awaiter (/Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:15:12)
    at ESLint8Plugin.invokeESLint (/Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:133:16)
    at ESLint8Plugin.<anonymous> (/Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:120:44)
    at step (/Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:44:23)
    at Object.next (/Applications/IntelliJ IDEA.app/Contents/plugins/JavaScriptLanguage/languageService/eslint/bin/eslint8-plugin.js:25:53)
Process finished with exit code -1
```

  
Versi Intellij

`IntelliJ IDEA 2022.1.4 (Ultimate Edition)`

Issue ini dikarenakan adanya update pada Eslint pada versi `8.23` .  
  
Solusi:

* Downgrade `Eslint` pada `package.json` ke versi `8.22`
    
* Update Intellij (Saya tidak mau memperpanjang lisensi 😅)
    

[https://youtrack.jetbrains.com/issue/WEB-57089/ESLint8.23-TypeError-this.libOptions.parse-is-not-a-function](https://youtrack.jetbrains.com/issue/WEB-57089/ESLint8.23-TypeError-this.libOptions.parse-is-not-a-function)