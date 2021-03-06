---
page_type: sample
products:
- office-365
- office-outlook
- ms-graph
languages:
- csharp
- aspx
description: "В этом примере для работы с данными используется клиентская библиотека .NET Microsoft Graph, а для аутентификации в конечной точке Azure AD версии 2.0"
extensions:
  contentType: samples 
  technologies:
  - Microsoft Graph
  services:
  - Office 365
  - Outlook
  - Groups
  createdDate: 8/4/2016 10:31:51 AM
---
# Пример фрагментов кода Microsoft Graph для ASP.NET 4.6

## Содержание

* [Предварительные требования](#prerequisites)
* [Регистрация приложения](#register-the-application)
* [Сборка и запуск примера](#build-and-run-the-sample)
* [Полезный код](#code-of-note)
* [Вопросы и комментарии](#questions-and-comments)
* [Участие](#contributing)
* [Дополнительные ресурсы](#additional-resources)

В этом примере представлены фрагменты кода, использующие Microsoft Graph для отправки электронной почты, управления группами и выполнения других стандартных задач из приложения ASP.NET MVC. Для работы с данными, возвращаемыми Microsoft Graph, используется [клиентский пакет SDK .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet). 

Для проверки подлинности в этом примере используется библиотека [Microsoft Authentication Library (MSAL)](https://www.nuget.org/packages/Microsoft.Identity.Client/). В пакете SDK MSAL предусмотрены функции для работы с [конечной точкой Azure AD версии 2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview), которая позволяет разработчикам создать единый поток кода для проверки подлинности как рабочих или учебных (Azure Active Directory), так и личных учетных записей Майкрософт.

Кроме того, в примере показано, как запрашивать маркеры пошагово. Эта функция поддерживается конечной точкой Azure AD версии 2.0. Пользователи предоставляют начальный набор разрешений при входе, но могут предоставить другие разрешения позже. Любой действительный пользователь может войти в это приложение, но администраторы могут позже предоставить разрешения, необходимые для определенных операций.

В примере используется [ПО промежуточного слоя ASP.NET OpenId Connect OWIN](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) для входа и при первом получении маркера. В примере также используется специальное ПО промежуточного слоя Owin для обмена кода авторизации на маркеры доступа и обновления после входа. Специальное ПО промежуточного слоя вызывает MSAL для создания URI запроса на авторизацию и обрабатывает перенаправления. Подробные сведения о предоставлении дополнительных разрешений см. в статье [Интеграция идентификатора Майкрософт и Microsoft Graph в веб-приложении с помощью OpenID Connect](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-v2).

> В этом примере используется ASP.NET MVC 4.6. Использование ASP.NET Core см. в любом из двух следующих примеров: — [Пример Microsoft Graph Connect для ASP.NET Core 2.1](https://github.com/microsoftgraph/aspnetcore-connect-sample) — [Разрешение входа пользователей в веб-приложения и вызов API с помощью платформы удостоверений Майкрософт для разработчиков](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

## Важное примечание о предварительной версии MSAL

Эту библиотеку можно использовать в рабочей среде. Для этой библиотеки мы предоставляем тот же уровень поддержки, что и для текущих библиотек рабочей среды. Мы можем внести изменения в API, формат внутреннего кэша и другие функциональные элементы, касающиеся этой предварительной версии библиотеки, которые вам потребуется принять вместе с улучшениями или исправлениями. Это может повлиять на ваше приложение. Например, в результате изменения формата кэша пользователям может потребоваться опять выполнить вход. При изменении API может потребоваться обновить код. Когда мы предоставим общедоступный выпуск, вам потребуется выполнить обновление до общедоступной версии в течение шести месяцев, так как приложения, написанные с использованием предварительной версии библиотеки, могут больше не работать.

## Необходимые условия

Для этого примера требуются следующие компоненты:  

  * [Visual Studio](https://www.visualstudio.com/en-us/downloads) 
  * [Учетная запись Майкрософт](https://www.outlook.com) или [учетная запись Office 365 для бизнеса](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account). Для выполнения административных операций требуется учетная запись администратора Office 365. Вы можете подписаться на [план Office 365 для разработчиков](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account), который включает ресурсы, необходимые для создания приложений.

## Регистрация веб-приложения

### Выбор клиента Azure AD для регистрации приложения

Сначала вам потребуется выполнить следующие действия:

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. Если ваша учетная запись имеется в нескольких клиентах Azure AD, выберите профиль в меню в правом верхнем углу страницы, а затем **переключите каталог**. Измените сеанс портала на нужный клиент Azure AD.

### Регистрация приложения

1. Перейдите на страницу [Регистрация приложений](https://go.microsoft.com/fwlink/?linkid=2083908) платформы удостоверений Майкрософт для разработчиков.
1. Выберите **Новая регистрация**.
1. После появления страницы **Регистрация приложения** введите сведения для регистрации приложения:
   - В разделе **Имя** введите понятное имя приложения, которое будет отображаться пользователям приложения.
   - В разделе **Поддерживаемые типы учетных записей** выберите вариант **Учетные записи в любом каталоге организации и личные учетные записи Майкрософт (например, Skype, Xbox, Outlook.com)**.
     > Обратите внимание, что существует несколько URI перенаправления. Их потребуется позже добавить из вкладки **Проверка подлинности** после успешного создания приложения.
1. Выберите **Зарегистрировать**, чтобы создать приложение.
1. На странице **Обзор** приложения найдите значение **Идентификатор приложения (клиент)** и запишите его для последующего использования. Для этого проекта вам потребуется настроить файл конфигурации Visual Studio.
1. На странице "Обзор" выберите раздел **Проверка подлинности**.
   - В разделе "URI перенаправления" выберите параметр **Веб** в поле со списком и введите указанные ниже URI переадресации.
       - `https://localhost:44300/`
       - `https://localhost:44300/signin-oidc`
   - В разделе **Дополнительные параметры** присвойте параметру **URL-адрес выхода** значение `https://localhost:44300/signout-oidc`
   - В разделе **Дополнительные параметры** | **Неявное предоставление** установите флажок **Токен идентификатора**, так как для этого примера требуется включить [поток неявного предоставления](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-implicit-grant-flow) для входа пользователя и вызова API.
1. Нажмите кнопку **Сохранить**.
1. На странице **Сертификаты и секреты** в разделе **Секреты клиента** нажмите кнопку **Новый секрет клиента**:
   - Введите описание ключа (например, `секрет приложения`).
   - Выберите, когда истекает срок действия ключа: **Через один год**, **Через два года** или **Никогда**.
   - После нажатия кнопки **Добавить** отобразится значение ключа. Скопируйте и сохраните значение в безопасном месте.
   - Этот ключ потребуется позже, чтобы настроить проект в Visual Studio. Это значение ключа больше не будет отображаться, и его невозможно получить каким-либо другим способом, поэтому запишите его, как только он появится на портале Azure.
 
## Сборка и запуск примера

1. Скачайте или клонируйте пример фрагментов кода Microsoft Graph для ASP.NET 4.6.

2. Откройте пример решения в Visual Studio.

3. В корневом каталоге файла Web.config замените заполнители **ida:AppId** и **ida:AppSecret** значениями, которые вы скопировали при регистрации приложения.

4. Нажмите клавишу F5 для сборки и запуска примера. При этом будут восстановлены зависимости пакета NuGet и откроется приложение.

   >Если при установке пакетов возникают ошибки, убедитесь, что локальный путь к решению не слишком длинный. Чтобы устранить эту проблему, переместите решение ближе к корню диска.

5. Войдите с помощью личной, рабочей или учебной учетной записи и предоставьте необходимые разрешения. 

6. Выберите категорию фрагментов, например "Пользователи", "Файлы" или "Почта". 

7. Выберите необходимую операцию. Обратите внимание на следующее:
  - Операции, для которых требуется аргумент (например, идентификатор), будут отключены до запуска фрагмента, позволяющего выбрать объект. 
  - Для некоторых фрагментов (помеченные *только для администраторов*) требуются платные разрешения, которые может предоставить только администратор. Чтобы запустить эти фрагменты, необходимо войти на портал Azure в качестве администратора. Затем используйте раздел *Разрешения API* страницы регистрации приложения, чтобы предоставить согласие на уровне администратора. Эта вкладка недоступна для пользователей, вошедших с помощью личных учетных записей.
  - Если вы вошли с помощью личной учетной записи, фрагменты, не поддерживающиеся для учетных записей Майкрософт, будут отключены.
   
Сведения об ответе отображаются в нижней части страницы.

### Как пример влияет на данные учетной записи

Этот пример создает, обновляет и удаляет объекты и данные (например, пользователей или файлы). Используя его, **вы можете изменить или удалить объекты и данные**, а также оставить артефакты данных. 

Чтобы этого избежать, обновляйте и удаляйте только те объекты, которые созданы примером. 


## Полезный код

- [Startup.Auth.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/App_Start/Startup.Auth.cs). Выполняет проверку подлинности для текущего пользователя и инициализирует кэш маркеров примера.

- [SessionTokenCache.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/TokenStorage/SessionTokenCache.cs). Хранит информацию о маркере пользователя. Вы можете заменить его на собственный кэш маркеров. Дополнительные сведения см. в статье [Кэширование маркеров доступа в мультитенантном приложении](https://azure.microsoft.com/en-us/documentation/articles/guidance-multitenant-identity-token-cache/).

- [SampleAuthProvider.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Helpers/SampleAuthProvider.cs). Реализует локальный интерфейс IAuthProvider и получает маркер доступа с помощью метода **AcquireTokenSilentAsync**. Вы можете заменить его на собственного поставщика услуг авторизации. 

- [SDKHelper.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Helpers/SDKHelper.cs). Инициализирует класс **GraphServiceClient** из [клиентской библиотеки .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet), используемой для взаимодействия с Microsoft Graph.

- Указанные ниже контроллеры содержат методы, использующие класс **GraphServiceClient** для создания и отправки вызовов в службу Microsoft Graph и обработки ответа.
  - [UsersController.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Controllers/UsersController.cs) 
  - [MailController.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Controllers/MailController.cs)
  - [EventsController.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Controllers/EventsController.cs) 
  - [FilesController.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Controllers/FilesController.cs)  
  - [GroupsController.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Controllers/GroupsController.cs) 

- Указанные ниже представления содержат пользовательский интерфейс примера.  
  - [Users.cshtml](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Views/Users/Users.cshtml)  
  - [Mail.cshtml](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Views/Mail/Mail.cshtml)
  - [Events.cshtml](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Views/Events/Events.cshtml) 
  - [Files.cshtml](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Views/Files/Files.cshtml)  
  - [Groups.cshtml](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Views/Groups/Groups.cshtml)

- Указанные ниже файлы содержат представления по умолчанию и частичное представление, которые используются для анализа и отображения данных Microsoft Graph в виде общих объектов (в этом примере). 
  - [ResultsViewModel.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Models/ResultsViewModel.cs)
  - [\_ResultsPartial.cshtml](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Views/Shared/_ResultsPartial.cshtml)  

- Указанные ниже файлы содержат код, используемый для поддержки дополнительных разрешений. В этом примере при входе пользователям предлагается принять начальный набор разрешений и отдельно выбрать разрешения администратора. 
  - [AdminController.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Controllers/AdminController.cs)
  - [OAuth2CodeRedeemerMiddleware.cs](/Graph-ASPNET-46-Snippets/Microsoft%20Graph%20ASPNET%20Snippets/Utils/OAuth2CodeRedeemerMiddleware.cs). Настраиваемое ПО промежуточного слоя, использующее код авторизации для маркеров обновления и доступа после входа. Дополнительные сведения о внедрении дополнительного согласия см. по адресу: https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-v2.

## Вопросы и комментарии

Мы будем рады узнать ваше мнение об этом примере. Вы можете отправлять нам свои вопросы и предложения на вкладке [Issues](https://github.com/microsoftgraph/aspnet-snippets-sample/issues) (Проблемы) этого репозитория.

Ваш отзыв важен для нас. Для связи с нами используйте сайт [Stack Overflow](http://stackoverflow.com/questions/tagged/microsoftgraph). Помечайте свои вопросы тегом \[MicrosoftGraph].

## Участие

Если вы хотите добавить код в этот пример, просмотрите статью [CONTRIBUTING.md](CONTRIBUTING.md).

Этот проект соответствует [правилам поведения Майкрософт, касающимся обращения с открытым кодом](https://opensource.microsoft.com/codeofconduct/). Дополнительные сведения см. в разделе [вопросов и ответов о правилах поведения](https://opensource.microsoft.com/codeofconduct/faq/). Если у вас возникли вопросы или замечания, напишите нам по адресу [opencode@microsoft.com](mailto:opencode@microsoft.com). 

## Дополнительные ресурсы

- [Другие примеры фрагментов кода Microsoft Graph](https://github.com/MicrosoftGraph?utf8=%E2%9C%93&query=snippets)
- [Общие сведения о Microsoft Graph](http://graph.microsoft.io)
- [Примеры кода решений для Office](http://dev.office.com/code-samples)
- [Центр разработки для Office](http://dev.office.com/)

## Авторские права
(c) Корпорация Майкрософт (Microsoft Corporation), 2016. Все права защищены.
