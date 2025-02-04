---
layout: post
title: Event Conversion Measurement API - более конфиденциальный способ измерения конверсии рекламы
subhead: >
  Новый веб-API, доступный в виде пробной версии источника, измеряет, когда клик на объявлении приводит к конверсии, без использования межсайтовых идентификаторов.
authors:
  - maudn
  - samdutton
hero: image/admin/wRrDtHNikUNqgdDewvYG.jpg
date: 2020-10-06
updated: 2020-05-04
tags:
  - blog
  - privacy
---

{% Banner 'caution', 'body' %} Conversion Measurement API будет переименован в *Attribution Reporting API* и получит больше функций.

- Если вы экспериментируете с [Conversion Measurement API](https://github.com/WICG/conversion-measurement-api/blob/3e0ef7d3cee8d7dc5a4b953e70cb027b0e13943b/README.md) в [Chrome 91](https://chromestatus.com/features/schedule) и ниже, то в этой публикации вы найдете более подробную информацию, варианты использования и инструкции к данному API.
- Если вас интересует следующая версия этого API (Attribution Reporting), которая будет доступна для экспериментов в Chrome (как пробная версия источника), [присоединяйтесь к списку рассылки](https://groups.google.com/u/1/a/chromium.org/g/attribution-reporting-api-dev), чтобы получать новости о доступных экспериментах.

{% endBanner %}

Для измерения эффективности рекламных кампаний рекламодатели и издатели должны знать, когда клик на объявлении или просмотр приводит к [конверсии](/digging-into-the-privacy-sandbox/#conversion), например к покупке товара или регистрации аккаунта. Традиционно это выполнялось с помощью **сторонних файлов cookie**. Теперь же Event Conversion Measurement API позволяет сопоставить событие на сайте издателя с последующей конверсией на сайте рекламодателя без использования механизмов узнавания пользователя на разных сайтах.

{% Banner 'info', 'body' %} **Нам важны отзывы об этом предложении!** Если у вас есть комментарии, [создайте тему](https://github.com/WICG/conversion-measurement-api/issues/) в репозитории предложения API. {% endBanner %}

{% Aside %} Этот API входит в тестовую среду конфиденциальности (Privacy Sandbox)—серию предложений для третьих лиц без использования сторонних файлов cookie и других механизмов межсайтового отслеживания. Обзор всех предложений см. в статье [«Углубляемся в тестовую среду конфиденциальности»](/digging-into-the-privacy-sandbox). {% endAside %}

## Глоссарий

- **Adtech-платформы**: компании, которые предоставляют программное обеспечение и инструменты, позволяющие брендам или агентствам таргетировать, доставлять и анализировать свою цифровую рекламу.
- **Рекламодатели**: компании, которые оплачивают рекламу.
- **Издатели**: компании, которые размещают рекламу на своих сайтах.
- **Конверсия по клику**: конверсия, связанная с кликом на объявлении.
- **Конверсия по показам**: конверсия, связанная с показом объявления (если пользователь не взаимодействует с объявлением, но затем всё же совершает конверсию).

## Кому нужно знать об этом APIadtech-платформам, рекламодателям и издателям

- **Adtech-платформам**, таким как **[автоматизированные платформы покупки рекламы](https://en.wikipedia.org/wiki/Demand-side_platform)**, этот API поможет поддерживать функции, которые сейчас зависят от сторонних файлов cookie. Если вы работаете над системами измерения конверсии, то [попробуйте демоверсию](#demo), [поэкспериментируйте с API](#experiment-with-the-api) и [поделитесь своим мнением](#share-your-feedback).
- **Рекламодатели и издатели, использующие собственный код для рекламы или измерения конверсии**, могут применять этот API вместо существующих методов.
- **Рекламодателям и издателям, которые используют adtech-платформы для рекламы или измерения конверсии**, не придется применять этот API напрямую, однако [разъяснение принципов этого API](#why-is-this-needed) может представлять для них интерес, особенно если их adtech-платформы захотят интегрировать этот API.

## Общие сведения об API

### Зачем это нужно?

Сегодня для измерения конверсии рекламы часто используются [сторонние файлы cookie](https://developer.mozilla.org/docs/Web/HTTP/Cookies#Third-party_cookies). **Однако браузеры ограничивают доступ к ним.**

Chrome планирует [отказаться от поддержки сторонних файлов cookie](https://blog.chromium.org/2020/01/building-more-private-web-path-towards.html) и [предлагает пользователям заблокировать их](https://support.google.com/chrome/answer/95647?co=GENIE.Platform%3DDesktop&hl=en). Safari [блокирует сторонние файлы cookie](https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/), Firefox [блокирует известные сторонние файлы cookie отслеживания](https://blog.mozilla.org/blog/2019/09/03/todays-firefox-blocks-third-party-tracking-cookies-and-cryptomining-by-default), а Edge [предлагает предотвращение отслеживания](https://support.microsoft.com/en-us/help/4533959/microsoft-edge-learn-about-tracking-prevention?ocid=EdgePrivacySettings-TrackingPrevention).

Сторонние файлы cookie становятся устаревшим решением. Взамен появляются **специализированные API** вроде этого, позволяющие выполнять те же задачи, что и сторонние файлы cookie, но с сохранением конфиденциальности.

**В чем отличие Event Conversion Measurement API от сторонних файлов cookie?**

- В отличие от файлов cookie, **он специально создан для измерения конверсий**. Это, в свою очередь, позволит браузерам реализовать усиленную защиту конфиденциальности.
- Он **более конфиденциальный** и затрудняет узнавание пользователя на двух разных сайтах верхнего уровня, например, для связывания профилей пользователей на стороне издателя и рекламодателя. Посмотрите, как [этот API сохраняет конфиденциальность пользователей](#how-this-api-preserves-user-privacy).

### Первая версия

Этот API находится на **ранней экспериментальной стадии**. В качестве пробной версии источника доступна **первая версия** API. В [будущих версиях](#use-cases) всё может существенно измениться.

### Только клики

Эта версия API поддерживает только **измерение конверсий по кликам**, а [измерение конверсий по показам](https://github.com/WICG/conversion-measurement-api/blob/main/event_attribution_reporting.md) находится в стадии общедоступной инкубации.

### Принцип работы

<figure>{% Img src="image/admin/Xn96AVosulGisR6Hoj4J.jpg", alt="Диаграмма: обзор этапов работы API измерения конверсии", width="800", height="496" %}</figure>

Этот API можно использовать с двумя типами ссылок (элементов `<a>`) для рекламы:

- ссылками в контексте **первой стороны**, такими как реклама в социальных сетях или на страницах результатов поиска;
- ссылками в **стороннем блоке iframe**, например на сайте издателя, который использует стороннего adtech-поставщика.

С помощью этого API такие исходящие ссылки можно настроить со специальными атрибутами для конверсии рекламы:

- пользовательскими данными, которые нужно связать с кликом на объявлении на стороне издателя, например идентификатором клика или идентификатором кампании;
- сайтом, для которого ожидается конверсия по данному объявлению;
- конечной отчетной точкой, которая должна получать уведомления об успешных конверсиях;
- конечной датой и временем, после которых конверсии для этого объявления больше не будут учитываться.

Когда пользователь нажимает на объявление, браузер на локальном устройстве пользователя регистрирует это событие вместе с конфигурацией конверсии и данными о кликах, выбранными в атрибутах измерения конверсии в элементе `<a>`.

Далее пользователь может посетить сайт рекламодателя и выполнить действие, которое рекламодатель или его поставщик рекламных технологий классифицирует как **конверсию**. В этом случае браузер пользователя свяжет клик на объявлении и событие конверсии.

Наконец, браузер планирует **отправку отчета о конверсии** в конечную точку, указанную в атрибутах элемента `<a>`. Этот отчет включает данные о клике на объявлении, который привел к конверсии, а также данные о самой конверсии.

Если для одного клика на объявлении регистрируется несколько конверсий, то планируется отправка соответствующего количества отчетов (до трех на каждый клик на объявлении).

Отчеты отправляются с задержкой: через несколько дней или даже недель после конверсии (о том, почему так происходит, см. в разделе [«Сроки отправки отчетов»](#report-timing)).

## Поддержка браузеров и похожие API

### Поддержка браузеров

Event Conversion Measurement API может поддерживаться:

- Как [пробная версия источника](/origin-trials/). Пробные версии источника позволяют использовать API для **всех посетителей** из определенного [источника](/same-site-same-origin/#origin)**. Чтобы опробовать API на конечных пользователях, необходимо зарегистрировать свой источник для пробной версии источника**. Подробнее о пробной версии источника см. в статье [«Использование API для измерения конверсии»](/using-conversion-measurement).
- При включении флагов в Chrome 86 и новее. Флаги включают API в браузере **отдельного пользователя**. **Флаги удобны при локальной разработке**.

Подробности о текущем статусе этой функции Chrome смотрите [здесь](https://chromestatus.com/features/6412002824028160).

### Стандартизация

Разработка этого API ведется открыто в группе Web Platform Incubator Community Group ([WICG](https://www.w3.org/community/wicg/)). Он доступен для экспериментов в Chrome.

### Похожие API

WebKit, движок веб-браузера Safari, предлагает похожее решение под названием [Private Click Measurement](https://github.com/privacycg/private-click-measurement). Над ним работает группа Privacy Community Group ([PrivacyCG](https://www.w3.org/community/privacycg/)).

## Как этот API сохраняет конфиденциальность пользователей

С помощью этого API можно измерять конверсии, сохраняя конфиденциальность пользователей, так как их нельзя будет узнать на разных сайтах. Это реализовано благодаря **ограничениям данных**, **зашумлению данных конверсии** и механизмам **управления временем отправки отчетов**.

Давайте рассмотрим подробнее, как работают эти механизмы и что они означают на практике.

### Ограничения данных

Далее под **данными о времени клика или просмотра** понимаются данные, доступные для `adtech.example` в тот момент, когда объявление показывается пользователю, а затем выполняется клик или просмотр. Данные, зарегистрированные во время конверсии, называются **данными времени конверсии**.

Давайте рассмотрим **издателя** `news.example` и **рекламодателя** `shoes.example`. Сторонние скрипты с **adtech-платформы** `adtech.example` работают на сайте издателя `news.example` и показывают объявления рекламодателя `shoes.example`. Кроме того, `shoes.example` включает скрипты `adtech.example` для обнаружения конверсий.

Что `adtech.example` может узнать о пользователях?

#### Со сторонними cookie

<figure>{% Img src="image/admin/kRpuY2r7ZSPtADz7e1P5.jpg", alt="Диаграмма: как сторонние cookie позволяют узнавать пользователей при их перемещении между сайтами", width="800", height="860" %}</figure>

`adtech.example` использует **сторонний файл cookie в качестве уникального межсайтового идентификатора** для **узнавания пользователя на разных сайтах**. Кроме того, `adtech.example` может получать доступ **одновременно** и к подробным данным о времени кликов или просмотров, и к подробным данным о времени конверсии, а также связывать эти данные между собой.

В результате `adtech.example` может отслеживать поведение пользователя на сайтах между просмотром рекламы, кликом и конверсией.

Поскольку `adtech.example`, вероятно, присутствует на большом количестве сайтов издателей и рекламодателей, а не только на `news.example` и `shoes.example`, поведение пользователя можно отслеживать в интернете.

#### С помощью Event Conversion Measurement API

<figure>{% Img src="image/admin/X6sfyeKGncVm0LJSYJva.jpg", alt="Диаграмма: как API позволяет измерять конверсии без узнавания пользователей при их перемещении между сайтами", width="800", height="643" %}<figcaption> «Идентификатор объявления» на диаграмме файлов cookie и «идентификатор клика»—это идентификаторы для привязки к подробным данным. На этой диаграмме используется название «идентификатор клика», потому что сейчас поддерживается только измерение конверсий по кликам.</figcaption></figure>

`adtech.example` не может использовать межсайтовый идентификатор и, следовательно, **не может узнавать пользователя на разных сайтах**.

- К клику на объявлении можно привязать 64-битный идентификатор.
- К событию конверсии можно привязать только 3 бита данных конверсии. 3 бита могут соответствовать целочисленному значению от 0 до 7. Это небольшой объем данных, но вполне достаточный для того, чтобы рекламодатели смогли понять, где лучше потратить свой рекламный бюджет в будущем (например, путем обучения моделей данных).

{% Aside %} Данные о кликах и конверсиях никогда не отображаются в среде JavaScript в одном и том же контексте. {% endAside %}

#### Без альтернативы сторонним cookie

Без таких альтернатив сторонним cookie, как Event Conversion Measurement API, конверсии невозможно атрибутировать: если `adtech.example` присутствует как на сайте издателя, так и на сайте рекламодателя, то сможет получить доступ к данным о времени клика или времени конверсии, но не сможет как-либо связать их между собой.

В этом случае конфиденциальность пользователей сохраняется, но рекламодатели не могут оптимизировать свои расходы на рекламу. Именно поэтому требуется такая альтернатива, как Event Conversion Measurement API.

### Зашумление данных конверсии

3 бита, собранные во время конверсии, **зашумляются**.

Например, в реализации Chrome зашумление данных работает так: в 5% случаев API передает случайное 3-битное значение вместо реальных данных конверсии.

Это защищает пользователей от атак на конфиденциальность. Злоумышленник, который попытается использовать данные нескольких конверсий для создания идентификатора, не сможет полностью доверять получаемым данным, что усложнит атаку.

Обратите внимание, что [истинное количество конверсий можно восстановить](/using-conversion-measurement/#(optional)-recover-the-corrected-conversion-count).

Вот краткий свод данных о кликах и конверсиях:

<div>
  <table data-alignment="top">
    <thead>
      <tr>
        <th>Данные</th>
        <th>Размер</th>
        <th>Пример</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Данные о клике (атрибут <code>impressiondata</code>)</td>
        <td>64 бит</td>
        <td>Идентификатор объявления или идентификатор клика</td>
      </tr>
      <tr>
        <td>Данные о конверсии</td>
        <td>3 бита с зашумлением</td>
        <td>Целое число от 0 до 7 в зависимости от типа конверсии: создание аккаунта, завершение оплаты и т. д.</td>
      </tr>
    </tbody>
  </table>
</div>

### Сроки отправки отчета

Если для одного клика на объявлении регистрируется несколько конверсий, то **отправляется соответствующее количество отчетов (до трех на каждый клик на объявлении)**.

Чтобы время конверсии нельзя было использовать для получения дополнительной информации от стороны конверсии и тем самым нарушать конфиденциальность пользователей, этот API не отправляет отчеты о конверсии сразу после конверсии. После первого клика на объявлении создается расписание **окон отправки отчетности** для этого клика. У каждого окна отправки отчетности есть крайний срок, и конверсии, зарегистрированные до этого крайнего срока, будут отправлены в конце этого окна.

Отчеты не обязательно отправляются именно в запланированную дату и время: если в запланированный момент для отправки отчета браузер не будет запущен, то отчет отправится при следующем запуске браузера, то есть иногда через несколько дней или даже недель.

По истечении срока действия (время клика + `impressionexpiry`) конверсии уже не засчитываются: `impressionexpiry` служит датой и временем отсечки, после которой конверсии больше не будут учитываться для данного объявления.

В Chrome планирование отправки отчетов работает так:

<div>
  <table data-alignment="top">
    <thead>
      <tr>
        <th><code>impressionexpiry</code></th>
        <th>В зависимости от времени конверсии отчет о конверсии (если браузер открыт) отправляется…</th>
        <th>Количество окон отправки отчетов</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>30 дней, значение по умолчанию и максимальное значение</td>
        <td>
          <ul>
            <li>через 2 дня после клика на объявлении</li>
            <li>или через 7 дней после клика на объявлении</li>
            <li>или через <code>impressionexpiry</code> = 30 дней после клика на объявлении.</li>
          </ul>
        </td>
        <td>3</td>
      </tr>
      <tr>
        <td>
<code>impressionexpiry</code> имеет значение от 7 до 30 дней</td>
        <td>
          <ul>
            <li>через 2 дня после клика на объявлении</li>
            <li>или через 7 дней после клика на объявлении</li>
            <li>или через <code>impressionexpiry</code> после клика на объявлении.</li>
          </ul>
        </td>
        <td>3</td>
      </tr>
      <tr>
        <td>
<code>impressionexpiry</code> имеет значение от 2 до 7 дней</td>
        <td>
          <ul>
            <li>через 2 дня после клика на объявлении</li>
            <li>или через <code>impressionexpiry</code> после клика на объявлении.</li>
          </ul>
        </td>
        <td>2</td>
      </tr>
      <tr>
        <td>
<code>impressionexpiry</code> имеет значение менее 2 дней</td>
        <td>
          <li>через 2 дня после клика на объявлении</li>
        </td>
        <td>1</td>
      </tr>
    </tbody>
  </table>
</div>

<figure>{% Img src="image/admin/bgkpW6Nuqs5q1ddyMG8X.jpg", alt="Хронология и расписание отправки отчетов", width="800", height="462" %}</figure>

Подробнее о расписании см. в разделе документации [«Отправка запланированных отчетов»](https://github.com/WICG/conversion-measurement-api#sending-scheduled-reports).

## Пример

{% Banner 'info', 'body' %} Чтобы увидеть, как всё это работает на практике, запустите [демоверсию](https://goo.gle/demo-event-level-conversion-measurement-api) ⚡️ и изучите соответствующий [код](https://github.com/GoogleChromeLabs/trust-safety-demo/tree/main/conversion-measurement). {% endBanner %}

Вот как API регистрирует конверсию и сообщает о ней. Обратите внимание, что поток событий между кликом и конверсией относится только к текущей версии API. Будущие версии этого API [могут отличаться](#use-cases).

### Клик на объявлении (шаги с 1 по 5)

<figure>{% Img src="image/admin/FvbacJL6u37XHuvQuUuO.jpg", alt="Диаграмма: клик на объявлении и хранилище кликов", width="800", height="694" %}</figure>

Элемент `<a>` загружается на сайт издателя в блок iframe с помощью `adtech.example`.

Разработчики adtech-платформы настроили элемент `<a>` с такими атрибутами измерения конверсии:

```html
<a
  id="ad"
  impressiondata="200400600"
  conversiondestination="https://advertiser.example"
  reportingorigin="https://adtech.example"
  impressionexpiry="864000000"
  href="https://advertiser.example/shoes07"
>
  <img src="/images/shoe.jpg" alt="shoe" />
</a>
```

Этот код определяет следующие атрибуты:

<div>
  <table data-alignment="top">
    <thead>
      <tr>
        <th>Атрибут</th>
        <th>Значение по умолчанию, максимум и минимум</th>
        <th>Пример</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>
<code>impressiondata</code> (обязательный): <b>64-битный</b> идентификатор, который привязывается к клику на объявлении.</td>
        <td>(нет значения по умолчанию)</td>
        <td>Динамически генерируемый идентификатор клика, например 64-битное целое число: <code>200400600</code>
</td>
      </tr>
      <tr>
        <td>
<code>conversiondestination</code> (обязательный): <b><a href="/same-site-same-origin/#site" noopener="">eTLD + 1</a></b>, где ожидается конверсия для данного объявления.</td>
        <td>(нет значения по умолчанию)</td>
        <td>
<code>https://advertiser.example</code>.<br> Если  <code>conversiondestination</code> имеет значение <code>https://advertiser.example</code>, то атрибуты будут регистрироваться для конверсии как на <code>https://advertiser.example</code>, так и на <code>https://shop.advertiser.example</code>.<br> То же самое происходит, если <code>conversiondestination</code> имеет значение <code>https://shop.advertiser.example</code>: конверсия учитывается как на <code>https://advertiser.example</code>, так и на <code>https://shop.advertiser.example</code>.</td>
      </tr>
      <tr>
        <td>
<code>impressionexpiry</code> (необязательный): время отсечки в миллисекундах, до которого конверсии могут регистрироваться для данного объявления.</td>
        <td>
<code>2592000000</code> = 30 дней (в миллисекундах).<br><br> Максимум: 30 дней (в миллисекундах).<br><br> Минимум: 2 дня (в миллисекундах).</td>
        <td>Через десять дней после клика: <code>864000000</code>
</td>
      </tr>
      <tr>
        <td>
<code>reportingorigin</code> (необязательный): конечный адрес для отправки отчетов о подтвержденных конверсиях.</td>
        <td>Источник верхнего уровня страницы, на которую добавлен ссылочный элемент.</td>
        <td><code>https://adtech.example</code></td>
      </tr>
      <tr>
        <td>
<code>href</code>: предполагаемый конечный адрес для клика на объявлении.</td>
        <td><code>/</code></td>
        <td><code>https://advertiser.example/shoes07</code></td>
      </tr>
    </tbody>
  </table>
</div>

{% Aside %} Несколько примечаний к примеру:

- В атрибутах API и в предложении API используется термин «показ» (impression), хотя пока что поддерживаются только клики. В будущих версиях API термины могут измениться.
- Объявление не обязательно должно находиться в блоке iframe, но данный пример строится именно на этом.

{% endAside %}

{% Aside 'gotchas' %}

- Для потоков с навигацией через `window.open` или `window.location` атрибуты не регистрируются.

{% endAside %}

Когда пользователь нажимает на объявление, он попадает на сайт рекламодателя. После подтверждения навигации браузер сохраняет объект, включающий в себя `impressiondata`, `conversiondestination`, `reportingorigin` и `impressionexpiry`:

```json
{
  "impression-data": "200400600",
  "conversion-destination": "https://advertiser.example",
  "reporting-origin": "https://adtech.example",
  "impression-expiry": 864000000
}
```

### Конверсия и планирование отправки отчетов (шаги с 6 по 9)

<figure>{% Img src="image/admin/2fFVvAwyiXSaSDp8XVXo.jpg", alt="Диаграмма: конверсия и планирование отправки отчетов", width="800", height="639" %}</figure>

Сразу после клика на объявлении или позднее, например на следующий день, пользователь посещает `advertiser.example`, просматривает ассортимент спортивной обуви, выбирает себе пару кроссовок и переходит к оплате. В этом случае `advertiser.example` добавит пиксель на страницу оформления заказа:

```html
<img
  height="1"
  width="1"
  src="https://adtech.example/conversion?model=shoe07&type=checkout&…"
/>
```

`adtech.example` получает этот запрос и классифицирует его как конверсию, а затем запрашивает у браузера регистрацию конверсии. Далее `adtech.example` сжимает все данные конверсии в 3 бита, получая целое число от 0 до 7, например число 2, если оно соответствует **действию оплаты**.

После этого `adtech.example` отправляет браузеру определенное перенаправление для регистрации конверсии:

```js
const conversionValues = {
  signup: 1,
  checkout: 2,
};

app.get('/conversion', (req, res) => {
  const conversionData = conversionValues[req.query.conversiontype];
  res.redirect(
    302,
    `/.well-known/register-conversion?conversion-data=${conversionData}`,
  );
});
```

{% Aside %}. URL-адреса `.well-known`—это особые URL-адреса, которые позволяют программным средствам и серверам легко находить необходимую информацию или ресурсы для сайта, например, на какой странице пользователь может [изменить свой пароль](/change-password-url/). Здесь `.well-known` используется только для того, чтобы браузер распознал специальный запрос на конверсию. Этот запрос фактически отменяется браузером. {% endAside %}

Браузер получает этот запрос и, обнаружив ссылку `.well-known/register-conversion`, выполняет следующие действия:

- поиск всех кликов на объявлениях в хранилище, которые соответствуют значению `conversiondestination` (так как конверсия получена по URL-адресу, который был зарегистрирован как `conversiondestination`, когда пользователь нажал на объявление), и определение клика на объявлении, который произошел на сайте издателя днем раньше;
- регистрацию конверсии для этого клика на объявлении.

Одной конверсии могут соответствовать и несколько кликов на объявлении: например, если пользователь нажмет на объявление для `shoes.example` как на `news.example`, так и на `weather.example`. В этом случае регистрируется несколько конверсий.

Теперь браузер знает, что ему необходимо сообщить adtech-серверу об этой конверсии, а именно об источнике `reportingorigin`, который указан как в элементе `<a>`, так и в запросе пикселя (`adtech.example`).

Для этого браузер планирует отправку **отчета** о конверсии—большого массива двоичных данных, содержащего данные о кликах (с сайта издателя) и данные о конверсиях (от рекламодателя). В этом примере пользователь совершил конверсию через день после клика. Таким образом, отправка отчета запланирована на следующий день, то есть через два дня после клика, если браузер запущен.

### Отправка отчета (шаги 10 и 11)

<figure>{% Img src="image/admin/Er48gVzK5gHUGdDHWHz1.jpg", alt="Диаграмма: браузер отправляет отчет", width="800", height="533" %}</figure>

По достижении запланированного срока отправки отчета браузер отправляет **отчет** о конверсии, а именно высылает HTTP POST источнику отчета, указанному в элементе `<a>` (`adtech.example`), например:

`https://adtech.example/.well-known/register-conversion?impression-data=200400600&conversion-data=2&credit=100`

В число параметров входят:

- данные, связанные с исходным кликом на объявлении (`impression-data`);
- данные, связанные с конверсией, [вероятно зашумленные](#noising-of-conversion-data);
- ценность конверсии, связанной с кликом: этот API реализует модель **атрибуции по последнему клику**, поэтому самому последнему подходящему клику на объявлении присваивается ценность 100, а всем остальным подходящим кликам на объявлении присваивается ценность 0.

Получив такой запрос, adtech-сервер может извлечь из него `impression-data` и `conversion-data`, то есть отчет о конверсии:

```json
{"impression-data": "200400600", "conversion-data": 3, "credit": 100}
```

### Последующие конверсии и истечение срока действия

Далее пользователь может снова совершить конверсию, например купить теннисную ракетку на сайте `advertiser.example` вместе с кроссовками. Тогда будет иметь место аналогичный поток событий:

- adtech-сервер отправит браузеру запрос на конверсию;
- браузер сопоставит эту конверсию с кликом на объявлении, запланирует отправку отчета и позднее отправит его adtech-серверу.

По истечении времени `impressionexpiry` любые конверсии для этого клика на объявлении перестанут учитываться, а сам клик на объявлении будет удален из хранилища браузера.

## Случаи применения

### Что поддерживается в текущей версии

- измерение конверсий по кликам: можно определять, какие клики на объявлениях приводят к конверсиям, и получать примерную информацию о конверсиях;
- сбор данных для оптимизации выбора рекламных мест, например с помощью моделей машинного обучения.

### Что не поддерживается в текущей версии

Следующие функции пока что не поддерживаются, но могут появиться в будущих версиях этого API или в [Aggregate](https://github.com/WICG/conversion-measurement-api/blob/master/AGGREGATE.md):

- измерение конверсий по показам;
- [несколько конечных точек приема отчетов](https://github.com/WICG/conversion-measurement-api/issues/29);
- [веб-конверсии, начатые в приложении для iOS/Android](https://github.com/WICG/conversion-measurement-api/issues/54);
- измерение прироста числа конверсий/инкрементность: измерение причинных различий в поведении конверсий путем измерения разницы между тестовой группой, которая увидела рекламу, и контрольной группой, которая ее не увидела;
- модели атрибуции, не основанные на последнем клике;
- варианты использования, требующие большего объема информации о конверсии, например, с детализацией по стоимости покупок или категориям товаров.

Прежде чем эти и другие функции начнут поддерживаться, **в API необходимо добавить дополнительные средства защиты конфиденциальности** (зашумление, уменьшение количества бит или иные ограничения).

Обсуждение дополнительных возможных функций ведется открыто в разделе [**вопросов** в репозитории предложений API](https://github.com/WICG/conversion-measurement-api/issues).

{% Aside %} Не нашли нужный вам вариант использования? Или хотите оставить отзыв об API? [Поделитесь своим мнением](#share-your-feedback). {% endAside %}

### Что еще может измениться в будущих версиях

- Данный API находится на ранней экспериментальной стадии. В будущих версиях API может претерпеть существенные изменения, некоторые из которых перечислены ниже. Так как цель этого API—измерять конверсии при сохранении конфиденциальности пользователей, в него могут вноситься любые изменения, способствующие более эффективному решению этой задачи.
- Название API и атрибутов могут меняться.
- Данные о кликах и конверсиях могут не требовать кодирования.
- Ограничение в 3 бита для данных о конверсии может увеличиться или уменьшиться.
- [Могут быть добавлены дополнительные функции](#what-is-not-supported-in-this-iteration) и **дополнительные средства защиты конфиденциальности** (зашумление, уменьшение количества бит или иные ограничения), если это потребуется для новых функций.

Чтобы сразу узнавать о новых функциях и участвовать в обсуждениях, изучайте [репозиторий](https://github.com/WICG/conversion-measurement-api/issues) предложения на GitHub и присылайте свои идеи.

## Попробуйте сами

### Запустите демоверсию

Запустите [демоверсию](https://goo.gle/demo-event-level-conversion-measurement-api) и обязательно следуйте инструкциям по началу работы с ней.

Пишите в Твиттер [@maudnals](https://twitter.com/maudnals?lang=en) или [@ChromiumDev](https://twitter.com/ChromiumDev), если у вас возникнут какие-либо вопросы о демоверсии!

### Поэкспериментируйте с API

Если вы планируете поэкспериментировать с API (локально или с конечными пользователями), изучите статью [«Использование API для измерения конверсий»](/using-conversion-measurement).

### Поделитесь своим мнением

**Ваш отзыв очень важен**, так как необходимые вам варианты использования могут быть реализованы в новых API для измерения конверсии, что принесет пользу и другим разработчикам.

- Чтобы сообщить об ошибке в реализации Chrome, [создайте сообщение об ошибке](https://bugs.chromium.org/p/chromium/issues/entry?status=Unconfirmed&components=Internals%3EConversionMeasurement&description=Chrome%20Version%3A%20%28copy%20from%20chrome%3A%2F%2Fversion%29%0AOS%3A%20%28e.g.%20Win10%2C%20MacOS%2010.12%2C%20etc...%29%0AURLs%20%28if%20applicable%29%20%3A%0A%0AWhat%20steps%20will%20reproduce%20the%20problem%3F%0A%281%29%0A%282%29%0A%283%29%0A%0AWhat%20is%20the%20expected%20result%3F%0A%0A%0AWhat%20happens%20instead%3F%0A%0AIf%20applicable%2C%20include%20screenshots%2Finfo%20from%20chrome%3A%2F%2Fconversion-internals%20or%20relevant%20devtools%20errors.%0A).
- Чтобы поделиться отзывами и обсудить варианты использования Chrome API, создайте новую задачу или участвуйте в обсуждении уже существующих в [репозитории предложений API](https://github.com/WICG/conversion-measurement-api/issues). Обсудить API WebKit/Safari и варианты его использования тоже можно в [репозитории предложений API](https://github.com/privacycg/private-click-measurement/issues).
- Чтобы обсудить варианты использования рекламы и обменяться мнениями с отраслевыми экспертами, вступайте в [группу Web Advertising Business Group](https://www.w3.org/community/web-adv/). Для обсуждения API WebKit/Safari присоединяйтесь к [группе Privacy Community Group](https://www.w3.org/community/privacycg/).

### Следите за новостями

- По мере сбора отзывов разработчиков и вариантов использования Event Conversion Measurement API будет развиваться. [Следите за обновлениями в репозитории](https://github.com/WICG/conversion-measurement-api/) предложения на GitHub.
- Следите также за развитием [Aggregate Conversion Measurement API](https://github.com/WICG/conversion-measurement-api/blob/master/AGGREGATE.md), который дополнит этот API.

*Большое спасибо за помощь и отзывы всем рецензентам, особенно Чарли Харрисону, Джону Делани, Майклу Клеберу и Кейси Баски.*

*Главное изображение взято у William Warby/@wawarby на [Unsplash](https://unsplash.com/photos/WahfNoqbYnM) и отредактировано.*
