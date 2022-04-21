# EmailMessageBuilder

## Быстрый старт

1. Создать модель представления для тела письма

```C#
public class TestEmail
{
    public long Id { get;set; } 
    public string Message { get; set; }
}
```

2. Создать шаблон тела письма в формате [liquid](https://github.com/scriban/scriban/blob/master/doc/language.md) </br>
Внутри {{}} указываются имена свойств модели, значения которых нужно отобразить.</br>
Для файла шаблона в Build Action устаносить **Embedded resource**.

```html
TestEmailTemplate.html 

<html>
<head>
</head>
<body>
    <div>Id: {{ Id }}</div>
    <div>Message: {{ Message }}</div>
</body>
</html>
``` 

3. Создать класс для регистрации шаблона и модели
```C#
public class TestEmailTemplate : Template<TestEmail>
{
    public override string TemplateName => "ProjectName.TemplatesFolder.TestEmailTemplate.html";
}
```
В TemplateName указывается имя embedded ресурса. </br>
Класс TestEmailTemplate должен лежать в той же сборке, в которой лежит embedded ресурс шаблона.

4. Зарегистрировать шаблон
```C#
dependencyService.RegisterMultiple<ITemplate, TestEmailTemplate>();
```

5. Используем IEmailMessageBuilder для формирования тела письма
```C#
public class Foo
{
    private readonly IEmailMessageBuilder messageBuilder;

    public Foo(IEmailMessageBuilder messageBuilder)
    {
        this.messageBuilder = messageBuilder;
    }

    public async Task SendEmailAsync()
    {
        var viewModel = new TestEmail
        {
            Id = 123,
            Message = "Test message"
        };

        string mailBody = await this.messageBuilder.BuildMessageAsync(viewModel);
    }
}

```

В mailBody получим 
```
Id: 123
Message: Test message
```

## Вложенные шаблоны
Решают проблему использования одного шаблона внутри другого, например, 
если есть базовый шаблон с шапкой и стилями, который нужно использовать в разных письмах.

При помощи функции include можно добавлять другие шаблоны в текущий.

Пример базового шаблона

```html
BaseTemplate.html

<html>
<head>
</head>
<body>
    {{ include 'content' }}
</body>
</html>
```

Пример вложенного шаблона
```html
TestEmailTemplate.html

<div>Id: {{ Id }}</div>
<div>Message: {{ Message }}</div>
```
Регистрация:
```C#
public class TestEmailTemplate : Template<TestEmail>
{
    public override string TemplateName => "ProjectName.TemplatesFolder.BaseTemplate.html";

    public override IEnumerable<(string key, string path)> Inclusions => new[]
    {
        ("content", "ProjectName.TemplatesFolder.TestEmailTemplate.html")
    };
}
```