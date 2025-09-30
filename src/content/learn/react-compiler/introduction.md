---
title: مقدمه
---

<Intro>
کامپایلر ری‌اکت یک ابزار جدید در زمان ساخت است که به‌طور خودکار اپ ری‌اکت شما را بهینه‌سازی می‌کند. این ابزار با جاوااسکریپت ساده کار می‌کند و [قوانین ری‌اکت](/reference/rules) را درک می‌کند، بنابراین نیازی به بازنویسی هیچ کدی برای استفاده از آن ندارید.
</Intro>

<YouWillLearn>

* کامپایلر ری‌اکت چه کاری انجام می‌دهد؟
* شروع کار با کامپایلر
* استراتژی‌های پذیرش تدریجی
* اشکال‌زدایی و رفع مشکل زمانی که کارها اشتباه پیش می‌روند
* استفاده از کامپایلر در کتابخانه ری‌اکت شما

</YouWillLearn>

<Note>
کامپایلر ری‌اکت در حال حاضر در مرحله کاندید انتشار (RC) است. اکنون به همه توصیه می‌کنیم که کامپایلر را امتحان کرده و بازخورد ارائه دهند. آخرین نسخه RC را می‌توانید با برچسب `@rc` پیدا کنید.
</Note>

## کامپایلر ری‌اکت چه کاری انجام می‌دهد؟ {/*what-does-react-compiler-do*/}

کامپایلر ری‌اکت به‌طور خودکار برنامه ری‌اکت شما را در زمان ساخت بهینه‌سازی می‌کند. ری‌اکت اغلب بدون بهینه‌سازی به اندازه کافی سریع است، اما گاهی اوقات نیاز دارید که کامپوننت‌ها و مقادیر را به‌صورت دستی ممویزه کنید تا برنامه شما پاسخگو باقی بماند. این ممویزه‌کردن دستی خسته‌کننده، مستعد خطا و نیازمند کد اضافی برای نگهداری است. کامپایلر ری‌اکت این بهینه‌سازی را به‌طور خودکار برای شما انجام می‌دهد و شما را از این بار ذهنی آزاد می‌کند تا بتوانید بر ساخت قابلیت‌ها تمرکز کنید.

### قبل از کامپایلر ری‌اکت {/*before-react-compiler*/}

بدون کامپایلر، شما باید به صورت دستی کامپوننت‌ها و مقادیر را برای بهینه‌سازی رندرهای مجدد، ممویزه کنید:

```js {expectedErrors: {'react-compiler': [4]}}
import { useMemo, useCallback, memo } from 'react';

const ExpensiveComponent = memo(function ExpensiveComponent({ data, onClick }) {
  const processedData = useMemo(() => {
    return expensiveProcessing(data);
  }, [data]);

  const handleClick = useCallback((item) => {
    onClick(item.id);
  }, [onClick]);

  return (
    <div>
      {processedData.map(item => (
        <Item key={item.id} onClick={() => handleClick(item)} />
      ))}
    </div>
  );
});
```


<Note>

این مموییزیشن دستی دارای یک خطای ظریف است که مموییزیشن را مختل می‌کند:

```js [[2, 1, "() => handleClick(item)"]]
<Item key={item.id} onClick={() => handleClick(item)} />
```

حتی اگر `handleClick` در `useCallback` قرار گرفته باشد، تابع پیکان `() => handleClick(item)` هر بار که کامپوننت رندر می‌شود، یک تابع جدید ایجاد می‌کند. این به این معناست که `Item` همیشه یک prop `onClick` جدید دریافت می‌کند و این باعث شکستن مموی‌سازی می‌شود.

کامپایلر ری‌اکت می‌تواند این را به‌درستی بهینه‌سازی کند، چه با تابع پیکان و چه بدون آن، و اطمینان حاصل کند که `Item` فقط زمانی که `props.onClick` تغییر می‌کند، مجدداً رندر می‌شود.

</Note>

### بعد از کامپایلر ری‌اکت {/*after-react-compiler*/}

با کامپایلر ری‌اکت، شما همان کد را بدون ممویزه‌سازی دستی می‌نویسید:

```js
function ExpensiveComponent({ data, onClick }) {
  const processedData = expensiveProcessing(data);

  const handleClick = (item) => {
    onClick(item.id);
  };

  return (
    <div>
      {processedData.map(item => (
        <Item key={item.id} onClick={() => handleClick(item)} />
      ))}
    </div>
  );
}
```

_[این مثال را در React Compiler Playground ببینید](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAogB4AOCmYeAbggMIQC2Fh1OAFMEQCYBDHAIA0RQowA2eOAGsiAXwCURYAB1iROITA4iFGBERgwCPgBEhAogF4iCStVoMACoeO1MAcy6DhSgG4NDSItHT0ACwFMPkkmaTlbIi48HAQWFRsAPlUQ0PFMKRlZFLSWADo8PkC8hSDMPJgEHFhiLjzQgB4+eiyO-OADIwQTM0thcpYBClL02xz2zXz8zoBJMqJZBABPG2BU9Mq+BQKiuT2uTJyomLizkoOMk4B6PqX8pSUFfs7nnro3qEapgFCAFEA)_

کامپایلر ری‌اکت به‌طور خودکار بهینه‌ترین مموی‌زیشن را اعمال می‌کند و اطمینان می‌دهد که اپلیکیشن شما فقط در مواقع ضروری دوباره رندر می‌شود.

<DeepDive>
#### کامپایلر ری‌اکت چه نوعی از memoization را اضافه می‌کند؟ {/*what-kind-of-memoization-does-react-compiler-add*/}

حافظه‌گذاری خودکار کامپایلر ری‌اکت عمدتاً بر **بهبود عملکرد به‌روزرسانی** (رندر مجدد کامپوننت‌های موجود) متمرکز است، بنابراین بر روی این دو مورد کاربرد تمرکز می‌کند:

1. **رد شدن از رندر مجدد زنجیره‌ای کامپوننت‌ها**
* رندر مجدد `<Parent />` باعث می‌شود بسیاری از کامپوننت‌ها در درخت کامپوننت آن دوباره رندر شوند، حتی اگر فقط `<Parent />` تغییر کرده باشد.
1. **صرف‌نظر از محاسبات پرهزینه خارج از ری‌اکت**
* به‌عنوان مثال، فراخوانی `expensivelyProcessAReallyLargeArrayOfObjects()` درون کامپوننت یا هوکی که به آن داده نیاز دارد.

#### بهینه‌سازی رندرهای مجدد {/*optimizing-re-renders*/}

ری‌اکت به شما اجازه می‌دهد رابط کاربری خود را به عنوان تابعی از وضعیت فعلی آن‌ها (به طور مشخص‌تر: props، state و context آن‌ها) بیان کنید. در پیاده‌سازی فعلی، زمانی که state یک کامپوننت تغییر می‌کند، ری‌اکت آن کامپوننت _و تمام فرزندان آن_ را دوباره رندر می‌کند — مگر اینکه شما از نوعی از memoization دستی با `useMemo()`، `useCallback()` یا `React.memo()` استفاده کرده باشید. به عنوان مثال، در مثال زیر، `<MessageButton>` هر بار که state `<FriendList>` تغییر کند، دوباره رندر خواهد شد.

```javascript
function FriendList({ friends }) {
  const onlineCount = useFriendOnlineCount();
  if (friends.length === 0) {
    return <NoFriends />;
  }
  return (
    <div>
      <span>{onlineCount} online</span>
      {friends.map((friend) => (
        <FriendListCard key={friend.id} friend={friend} />
      ))}
      <MessageButton />
    </div>
  );
}
```
[_این مثال را در محیط آزمایشی کامپایلر React ببینید_](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAYjHgpgCYAyeYOAFMEWuZVWEQL4CURwADrEicQgyKEANnkwIAwtEw4iAXiJQwCMhWoB5TDLmKsTXgG5hRInjRFGbXZwB0UygHMcACzWr1ABn4hEWsYBBxYYgAeADkIHQ4uAHoAPksRbisiMIiYYkYs6yiqPAA3FMLrIiiwAAcAQ0wU4GlZBSUcbklDNqikusaKkKrgR0TnAFt62sYHdmp+VRT7SqrqhOo6Bnl6mCoiAGsEAE9VUfmqZzwqLrHqM7ubolTVol5eTOGigFkEMDB6u4EAAhKA4HCEZ5DNZ9ErlLIWYTcEDcIA)

کامپایلر ری‌اکت به‌طور خودکار معادل مموی‌سازی دستی را اعمال می‌کند و اطمینان می‌دهد که فقط بخش‌های مرتبط یک اپلیکیشن با تغییر state دوباره رندر می‌شوند، که گاهی به آن "واکنش‌پذیری دقیق" گفته می‌شود. در مثال بالا، کامپایلر ری‌اکت تعیین می‌کند که مقدار بازگشتی `<FriendListCard />` می‌تواند حتی با تغییر `friends` مجدداً استفاده شود و می‌تواند از بازسازی این JSX و همچنین از رندر مجدد `<MessageButton>` با تغییر count جلوگیری کند.

#### محاسبات پرهزینه نیز به‌صورت memoized ذخیره می‌شوند {/*expensive-calculations-also-get-memoized*/}

کامپایلر ری‌اکت می‌تواند به‌طور خودکار محاسبات پرهزینه‌ای که در طول رندر استفاده می‌شوند را به‌صورت خودکار به خاطر بسپارد:

```js
// **Not** memoized by React Compiler, since this is not a component or hook
function expensivelyProcessAReallyLargeArrayOfObjects() { /* ... */ }

// Memoized by React Compiler since this is a component
function TableContainer({ items }) {
  // This function call would be memoized:
  const data = expensivelyProcessAReallyLargeArrayOfObjects(items);
  // ...
}
```
[این مثال را در محیط آزمایشی کامپایلر React ببینید](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAejQAgFTYHIQAuumAtgqRAJYBeCAJpgEYCemASggIZyGYDCEUgAcqAGwQwANJjBUAdokyEAFlTCZ1meUUxdMcIcIjyE8vhBiYVECAGsAOvIBmURYSonMCAB7CzcgBuCGIsAAowEIhgYACCnFxioQAyXDAA5gixMDBcLADyzvlMAFYIvGAAFACUmMCYaNiYAHStOFgAvk5OGJgAshTUdIysHNy8AkbikrIKSqpaWvqGIiZmhE6u7p7ymAAqXEwSguZcCpKV9VSEFBodtcBOmAYmYHz0XIT6ALzefgFUYKhCJRBAxeLcJIsVIZLI5PKFYplCqVa63aoAbm6u0wMAQhFguwAPPRAQA+YAfL4dIloUmBMlODogDpAA)

با این حال، اگر `expensivelyProcessAReallyLargeArrayOfObjects` واقعاً یک تابع پرهزینه است، ممکن است بخواهید پیاده‌سازی memoization آن را خارج از ری‌اکت در نظر بگیرید، زیرا:

- کامپایلر ری‌اکت فقط کامپوننت‌ها و هوک‌های ری‌اکت را به خاطر می‌سپارد، نه هر تابع.
- حافظه‌گذاری کامپایلر ری‌اکت بین چندین کامپوننت یا هوک مشترک نیست.

بنابراین اگر `expensivelyProcessAReallyLargeArrayOfObjects` در کامپوننت‌های مختلف استفاده شود، حتی اگر همان آیتم‌ها به‌طور دقیق ارسال شوند، آن محاسبه پرهزینه به‌طور مکرر اجرا خواهد شد. ما توصیه می‌کنیم ابتدا [پروفایل‌گیری](reference/react/useMemo#how-to-tell-if-a-calculation-is-expensive) کنید تا ببینید آیا واقعاً آن‌قدر پرهزینه است یا خیر، قبل از اینکه کد را پیچیده‌تر کنید.
</DeepDive>

## آیا باید کامپایلر را امتحان کنم؟ {/*should-i-try-out-the-compiler*/}

ما همه را تشویق می‌کنیم که استفاده از کامپایلر ری‌اکت را آغاز کنند. در حالی که کامپایلر هنوز یک افزودنی اختیاری برای ری‌اکت است، در آینده ممکن است برخی قابلیت‌ها برای عملکرد کامل به کامپایلر نیاز داشته باشند.

### آیا استفاده از آن امن است؟ {/*is-it-safe-to-use*/}

کامپایلر ری‌اکت اکنون در مرحله RC است و به‌طور گسترده‌ای در محیط تولید آزمایش شده است. در حالی که در شرکت‌هایی مانند Meta در تولید استفاده شده است، استفاده از کامپایلر در محیط تولید برای برنامه شما به سلامت کدبیس شما و میزان رعایت [قوانین ری‌اکت](/reference/rules) بستگی دارد.

## چه ابزارهای بیلدی پشتیبانی می‌شوند؟ {/*what-build-tools-are-supported*/}

کامپایلر ری‌اکت می‌تواند در [چندین ابزار ساخت](/learn/react-compiler/installation) مانند Babel، Vite، Metro و Rsbuild نصب شود.

کامپایلر ری‌اکت در اصل یک پوشش سبک برای پلاگین Babel است که حول کامپایلر اصلی ساخته شده و به گونه‌ای طراحی شده که از خود Babel جدا باشد. در حالی که نسخه پایدار اولیه کامپایلر عمدتاً به عنوان یک پلاگین Babel باقی خواهد ماند، ما با تیم‌های swc و [oxc](https://github.com/oxc-project/oxc/issues/10048) همکاری می‌کنیم تا پشتیبانی درجه یک برای کامپایلر ری‌اکت ایجاد کنیم تا در آینده نیازی به افزودن Babel به خطوط بیلد خود نداشته باشید.

کاربران Next.js می‌توانند کامپایلر ری‌اکت فراخوانی‌شده توسط swc را با استفاده از [v15.3.1](https://github.com/vercel/next.js/releases/tag/v15.3.1) و نسخه‌های بالاتر فعال کنند.

## چه کاری باید در مورد useMemo، useCallback و React.memo انجام دهم؟ {/*what-should-i-do-about-usememo-usecallback-and-reactmemo*/}

کامپایلر ری‌اکت به‌طور خودکار و با دقت و جزئیات بیشتری نسبت به [`useMemo`](/reference/react/useMemo)، [`useCallback`](/reference/react/useCallback) و [`React.memo`](/reference/react/memo) مموی‌سازی انجام می‌دهد. اگر تصمیم بگیرید که مموی‌سازی دستی را حفظ کنید، کامپایلر ری‌اکت آن‌ها را تحلیل کرده و تعیین می‌کند که آیا مموی‌سازی دستی شما با مموی‌سازی خودکار استنباط‌شده مطابقت دارد یا خیر. اگر مطابقتی وجود نداشته باشد، کامپایلر تصمیم می‌گیرد که بهینه‌سازی آن کامپوننت را متوقف کند.

این کار از روی احتیاط انجام می‌شود زیرا یک الگوی ضد معمول در مموی‌سازی دستی، استفاده از آن برای درستی است. این به این معناست که برنامه شما به مقادیر خاصی که مموی‌سازی شده‌اند برای عملکرد صحیح وابسته است. به عنوان مثال، برای جلوگیری از یک حلقه بی‌نهایت، ممکن است برخی مقادیر را مموی‌سازی کرده باشید تا از اجرای یک فراخوانی `useEffect` جلوگیری کنید. این کار قوانین ری‌اکت را نقض می‌کند، اما از آنجا که حذف خودکار مموی‌سازی دستی توسط کامپایلر می‌تواند بالقوه خطرناک باشد، کامپایلر به جای آن از این کار صرف‌نظر می‌کند. شما باید مموی‌سازی دستی خود را حذف کرده و اطمینان حاصل کنید که برنامه شما همچنان به درستی کار می‌کند.

## کامپایلر ری‌اکت را امتحان کنید {/*try-react-compiler*/}

این بخش به شما کمک می‌کند تا با کامپایلر ری‌اکت شروع کنید و بفهمید چگونه به‌طور مؤثر از آن در پروژه‌های خود استفاده نمایید.

* **[نصب](/learn/react-compiler/installation)** - ری‌اکت کامپایلر را نصب کرده و آن را برای ابزارهای بیلد خود پیکربندی کنید.
* **[سازگاری نسخه ری‌اکت](/reference/react-compiler/target)** - پشتیبانی از ری‌اکت ۱۷، ۱۸ و ۱۹
* **[Configuration](/reference/react-compiler/configuration)** - سفارشی‌سازی کامپایلر برای نیازهای خاص شما
* **[پذیرش تدریجی](/learn/react-compiler/incremental-adoption)** - استراتژی‌هایی برای پیاده‌سازی تدریجی کامپایلر در کدبیس‌های موجود
* **[اشکال‌زدایی و رفع مشکلات](/learn/react-compiler/debugging)** - شناسایی و رفع خطاها هنگام استفاده از کامپایلر
* **[کامپایل کتابخانه‌ها](/reference/react-compiler/compiling-libraries)** - بهترین روش‌ها برای ارائه کد کامپایل‌شده
* **[مرجع API](/reference/react-compiler/configuration)** - مستندات دقیق از تمامی گزینه‌های پیکربندی

## منابع اضافی {/*additional-resources*/}

علاوه بر این اسناد، پیشنهاد می‌کنیم برای اطلاعات و بحث‌های بیشتر دربارهٔ کامپایلر، به [React Compiler Working Group](https://github.com/reactwg/react-compiler) مراجعه کنید.

