---
layout: post
title:  "داشبورد مدیریتی برای ReactJS با تم متریال دیزاین"
date:   2018-05-01 15:50:00 +0430
comments: true
tags:
    - react
    - reactjs
    - material design
---
همین اول بگم از سر بی موضوعی این پست رو می‌نویسم، شاید استارتی شد برای ادامه!

من جدیدا به [ReactJS](https://reactjs.org/) روی آوردم، هنوز خیلی مسلط نشدم که بخوام مطالب پیچیده در موردش بنویسم اما اولین چالشی که باهاش داشتم، پیاده سازی یک پنل مدیریتی تر و تمیز و شیک بود که routeها، منوی راهبری و … توش پیاده شده باشه و من بتونم با خیال راحت کار توسعه رو انجام بدم. از اونجایی که ارادت خاصی به material design داشتم، کمی سرچ کردم و به [material-ui](https://material-ui-next.com) رسیدم.

در حال مطالعه و تست کامپوننت‌های مختلفش بودم که رسیدم به این پروژه! پروژه رو ستاپ کردم و اولین مشکل، عدم پشتیبانی از RTL بود و البته فونت مناسب برای زبان فارسی. از اونجایی که خود material-ui به خوبی از RTL پشتیبانی می‌کرد، با توجه به [مستندات خودش](https://material-ui-next.com/guides/right-to-left/)  ، تغییراتی در [پروژه‌ی اصلی](https://github.com/creativetimofficial/material-dashboard-react) دادم و کل داشبور رو RTL کردم و [فونت زیبای وزیر](https://github.com/rastikerdar/vazir-font) که کار آقای راستیکردار بود هم بهش اضافه کردم. شما می‌تونید [پروژه رو از این آدرس ببینید](https://github.com/moshtaghi/material-dashboard-react-rtl) و اگر دوست داشتید استفاده کنید. یا اینکه ادامه مطلب رو بخونید تا با چگونگی انجامش آشنا شید.

####راست‌چین کردن material-ui

مرحله ۱) ابتدا از فایل `public/index.html` را باز کنید و به `body` و `html` پروپرتی `direction` رو با مقدار rtl بدید.

```html
<html lang="fa_IR" direction="rtl">
...
<body direction="rtl">
```

[مشاهده تغییرات کد در gitHub](https://github.com/moshtaghi/material-dashboard-react-rtl/commit/65895039ac6a58b925f88b943ae7a1bf5ebce3b1#diff-528c3923d718a8860f5d8c05c3931c55)

مرحله ۲) حالا باید برای پروژه یک theme تعریف کنیم و `direction: rtl` رو در اون اعمال کنیم. برای اینکار فایل `src/layouts/Dashboard/Dashboard.jsx` رو باز و این کد‌ها رو به جاهایی که باید اضافه می‌کنیم.

```jsx
import { MuiThemeProvider, createMuiTheme } from 'material-ui/styles';
```

در کد بالا که بطور مشخص لوازمی که برای ایجاد قالب نیاز داریم رو import کردیم.

حالا باید با استفاده از متد `createMuiTheme` کانفیگ قالب خودمون رو ایجاد کنیم. به این شکل:

```jsx
const theme = createMuiTheme({
  direction: 'rtl'
});
```

و در پایان باید با استفاده از کامپوننت `MuiThemeProvider` قالب خودمون (همون ثابت `theme`) رو به کل پروژه اعمال کنیم.

```jsx
return (
	<MuiThemeProvider theme={theme}>
	...
	</MuiThemeProvider>
);
```

[مشاهده تغییرات کد در gitHub](https://github.com/moshtaghi/material-dashboard-react-rtl/commit/3882d49c228834c5463c85cf17e2b51899ff620d#diff-db6c09be1f26362d8e8eab0a45371845)

دلیل اینکه قالب رو به این کامپوننت اضافه کردیم این بود که در واقع این کامپوننت پس از کامپوننت‌های مربوط به route مادر همه‌ی کامپوننت‌هاست و بقیه تو دل این render می‌شن. پس با این حساب وقتی قالب رو به این اعمال کنیم، یعنی به همه اعمال کردیم.

مرحله ۳) خودشون گفتن از پلاگین jss-rtl استفاده کنیم. پس اول بصورت dependency نصبش می‌کنیم

```Bash
yarn add jss-rtl
```

بعد می‌ریم تو همون فایل بالا و این تغییرات رو یکی یکی می‌دیم بره...

اول import کردن چیزایی که لازمه برای اینکار

```jsx
import { create } from 'jss';
import rtl from 'jss-rtl';
import JssProvider from 'react-jss/lib/JssProvider';
import { createGenerateClassName, jssPreset } from 'material-ui/styles';
```

حالا باید jss رو کانفیگ کنیم، به این شکل:

```jsx
// Configure JSS
const jss = create({ plugins: [...jssPreset().plugins, rtl()] });

// Custom Material-UI class name generator.
const generateClassName = createGenerateClassName();
```

در انتها نیز با استفاده از کامپوننت `JssProvider` کانفیگ رو به کامپوننت App که در اینجا همون کامپوننت مادر باشه اعمال می‌کنیم.

```jsx
return (
    <MuiThemeProvider theme={theme}>
        <JssProvider jss={jss} generateClassName={generateClassName}>
            ...
        </JssProvider>
    </MuiThemeProvider>
);
```

[مشاهده تغییرات کد در gitHub](https://github.com/moshtaghi/material-dashboard-react-rtl/commit/6930a89165f674371867c86cef225348ef97b2b8#diff-db6c09be1f26362d8e8eab0a45371845)

####اصلاح استایل‌های مربوط به داشبورد

می‌شه گفت بطور کامل مراحل راست‌چین کردن کتابخونه material-ui رو انجام دادیم، اما خروجی هنوز طوری که باید باشه نیست. دلیل این موضوع انگولک‌هایی هست که در پروژه‌ی داشبورد انجام دادن و استایل‌های کاستوم بهش اعمال کردن.

ما برای بهتر کردن خروجی باید بصورت دستی، استایل‌های مربوط به کامپوننت‌های داشبورد رو که در مسیر `src/assets/jss` هست رو تغییر بدیم.

پس فایل استایلی که مربوط به کامپوننت مادر (`src/assets/jss/material-dashboard-react/appStyle.jsx`) باشه رو باز می‌کنیم و ...

```jsx
const appStyle = theme => ({
  wrapper: {
    direction: "rtl",
      ...
  },
  mainPanel: {
      ...
    float: "left",
      ...
  },
});
```

[مشاهده تغییرات کد در gitHub](https://github.com/moshtaghi/material-dashboard-react-rtl/commit/4768d5166512877eb26c90c207ef1fc264086bf5#diff-3ee9c081bfd380b7245c611eca1a7c13)

####افزودن فونت وزیر

برای اینکار از cdn استفاده می‌کنیم. پس ابتدا فایل `public/index.html` را باز کنید و کد زیر رو در اون قرار بدید.

```html
<link href="//cdn.rawgit.com/rastikerdar/vazir-font/v18.0.0/dist/font-face.css" rel="stylesheet" type="text/css" />
```

[مشاهده تغییرات کد در gitHub](https://github.com/moshtaghi/material-dashboard-react-rtl/commit/9085825b2b60f383d1e4173ca254ff77d4aaf58b#diff-528c3923d718a8860f5d8c05c3931c55)

اما نکته خوب اینکه material-ui برای اعمال font face یک راه بسیار خوب پیش پا قرار داده و اون استفاده از همون custom theme خودمون.

ثابت theme رو که قبلا تعریف کرده بودیم رو بصورت زیر تغییر می‌دیم.

```jsx
const theme = createMuiTheme({
  direction: 'rtl',
  typography: {
    fontFamily: '"Vazir", sans-serif'
  }
});
```

[مشاهده تغییرات کد در gitHub](https://github.com/moshtaghi/material-dashboard-react-rtl/commit/9085825b2b60f383d1e4173ca254ff77d4aaf58b#diff-db6c09be1f26362d8e8eab0a45371845)

الان باید تمامی متون در تمامی کامپوننت‌ها با فونت زیرای وزیر نمایش داده بشن، اما اون انگولک‌ها دست از سرمون بر نداشته هنوز. پس فایل `src/assets/jss/material-dashboard-react.jsx` رو باز می‌کنیم و ثابت `defaultFont` رو بصورت زیر تغییر می‌دیم

```jsx
const defaultFont = {
  fontWeight: "300",
  lineHeight: "1.5em"
};
```

[مشاهده تغییرات کد در gitHub](https://github.com/moshtaghi/material-dashboard-react-rtl/commit/9085825b2b60f383d1e4173ca254ff77d4aaf58b#diff-3c43498b7342c72ed30bfcc6d9b33fa8)

####همه چیز که نباید راست‌چین باشه!

یک ریزه‌کاری هم مونده، و اون اینکه direction چارت‌ها رو به همون شکل چپ‌چین نمایش بدیم تا بهتر دیده شن. پس فایل `src/assets/jss/material-dashboard-react/chartCardStyle.jsx` رو باز می‌کنیم و ...

```jsx
...
const chartCardStyle = {
  card,
  cardHeader: {
      ...
    direction: "ltr",
      ...
  }
```

[مشاهده تغییرات کد در gitHub](https://github.com/moshtaghi/material-dashboard-react-rtl/commit/e87ca566057d1d98353f8d3a7cfeb1bd7da163ca#diff-f0c18de7e6da7e2fbe5e348ad9171a48)

همین!

اگر از عبارات فارسی استفاده کنید می‌بینید که به چه زیبایی نمایش داده می‌شن.

ممنون که این مطلب رو مطالعه کردید. سعی می‌کنم مطالب بعدی پربارتر و تخصصی‌تر باشه.
