---
title: "[Error] npx create-react-app"
seoTitle: "npx create-react-app error"
seoDescription: "npx create-react-app error"
datePublished: Tue Jan 21 2025 15:36:04 GMT+0000 (Coordinated Universal Time)
cuid: cm66n0oxd000b09lb69yx8gu3
slug: error-npx-create-react-app
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/O_Xy25Dj7Mo/upload/857d641313b8e971d5d750198efa52d0.jpeg
tags: error

---

```bash
npx create-react-app frontend --template typescript

Installing packages. This might take a couple of minutes.
Installing react, react-dom, and react-scripts with cra-template-typescript...


added 1323 packages in 1m

267 packages are looking for funding
  run `npm fund` for details

Initialized a git repository.

Installing template dependencies using npm...
npm error code ERESOLVE
npm error ERESOLVE unable to resolve dependency tree
npm error
npm error While resolving: frontend@0.1.0
npm error Found: react@19.0.0
npm error node_modules/react
npm error   react@"^19.0.0" from the root project
npm error
npm error Could not resolve dependency:
npm error peer react@"^18.0.0" from @testing-library/react@13.4.0
npm error node_modules/@testing-library/react
npm error   @testing-library/react@"^13.0.0" from the root project
npm error
npm error Fix the upstream dependency conflict, or retry
npm error this command with --force or --legacy-peer-deps
npm error to accept an incorrect (and potentially broken) dependency resolution.
```

Ini dikarenakan ada dependency issue dari `testing-library/react@13.4.0` yang mana library ini memerlukan React versi 18 sedangkan project yang akan dibuat menggunakan React 19. Secara dokumentasi pada React.js 19 pembuatan project dengan menggunakan `npx create-react-app` sudah tidak direkomendasikan dan menyaranakan penggunakan React menggunakan framework seperti `Next.js` atau `Remix`. Jika masih ingin membuat project React tanpa menggunakan framework beberapa cara yang dapat dilakukan

Install menggunakan `Vite`

`npm create vite@latest frontend -- --template react` atau  
`npm create vite@latest frontend -- --template react-ts` (React dengan Typescript)