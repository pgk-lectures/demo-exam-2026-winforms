# Практическая работа: ООО «Обувь» на Windows Forms

В этой работе мы создадим приложение C#, которое подключается к базе `shoe_store` из MySQL.

Интерфейс создаётся мышкой в Visual Studio Designer. На C# пишется логика форм и запросы к базе. Figma, картинки интерфейса и готовые скриншоты не используются.

Перед началом выполните работу [База данных ООО «Обувь» в MySQL Workbench](https://github.com/pgk-lectures/demo-exam-2026-mysql-workbench).

## Что получится

- форма входа и кнопка гостевого режима;
- каталог товаров в `DataGridView`;
- разные права гостя, клиента, менеджера и администратора;
- поиск, фильтр по поставщику и сортировка по остатку;
- добавление, изменение и удаление товара администратором;
- просмотр заказов менеджером и администратором;
- изменение статуса и удаление заказа администратором.

# Часть 1. Создание проекта

## Шаг 1. Создайте проект

1. Запустите Visual Studio 2022.
2. Нажмите **Создать проект**.
3. Найдите **Windows Forms App** с языком C#.
4. Нажмите **Далее**.
5. Имя проекта: `ShoeStoreApp`.
6. Выберите **.NET 8.0**.
7. Нажмите **Создать**.
8. Удалите стандартную `Form1`, потому что первой будет форма входа.

## Шаг 2. Установите MySql.Data

1. В **Обозревателе решений** щёлкните правой кнопкой по проекту.
2. Выберите **Управление пакетами NuGet**.
3. Откройте вкладку **Обзор**.
4. Найдите `MySql.Data`.
5. Выберите пакет от Oracle.
6. Нажмите **Установить** и подтвердите установку.

Пакет добавляет классы `MySqlConnection`, `MySqlCommand` и `MySqlDataAdapter`.

# Часть 2. Формы и элементы

## Шаг 3. Создайте четыре формы

Для каждой формы щёлкните правой кнопкой по проекту и выберите **Добавить → Форма Windows Forms**.

Создайте:

```text
LoginForm.cs
ProductsForm.cs
ProductEditForm.cs
OrdersForm.cs
```

У всех форм задайте:

| Свойство | Значение | Что делает |
| --- | --- | --- |
| StartPosition | CenterScreen | Открывает окно по центру |
| Font | Times New Roman, 11pt | Общий шрифт задания |
| BackColor | 255; 255; 255 | Белый фон |

## Шаг 4. Соберите LoginForm

Добавьте элементы:

| Тип | (Name) | Text | Дополнительные свойства |
| --- | --- | --- | --- |
| Label | `lblTitle` | ООО «Обувь» | Font: 22pt, Bold |
| Label | `lblLogin` | Логин | — |
| TextBox | `txtLogin` | — | — |
| Label | `lblPassword` | Пароль | — |
| TextBox | `txtPassword` | — | UseSystemPasswordChar: True |
| Button | `btnLogin` | Войти | BackColor: 0; 250; 154 |
| Button | `btnGuest` | Войти как гость | — |

Расположите элементы сверху вниз. Размер формы можно установить `520, 380`.

## Шаг 5. Соберите ProductsForm

Размер формы: `1250, 760`.

Добавьте:

| Тип | (Name) | Назначение |
| --- | --- | --- |
| Label | `lblTitle` | Заголовок «Каталог товаров» |
| Label | `lblCurrentUser` | ФИО и роль справа сверху |
| Panel | `pnlManagement` | Панель поиска и фильтров |
| TextBox | `txtSearch` | Поисковая строка внутри панели |
| ComboBox | `cboSupplier` | Фильтр поставщика |
| ComboBox | `cboSort` | Сортировка остатка |
| DataGridView | `dgvProducts` | Список товаров |
| Button | `btnRefresh` | Обновить |
| Button | `btnAdd` | Добавить товар |
| Button | `btnEdit` | Изменить товар |
| Button | `btnDelete` | Удалить товар |
| Button | `btnOrders` | Заказы |
| Button | `btnLogout` | Выйти |

Для `dgvProducts` установите:

| Свойство | Значение | Что делает |
| --- | --- | --- |
| ReadOnly | True | Запрещает менять базу прямо в ячейке |
| AllowUserToAddRows | False | Убирает пустую нижнюю строку |
| MultiSelect | False | Разрешает выбрать одну строку |
| SelectionMode | FullRowSelect | Выделяет всю строку |
| AutoSizeColumnsMode | Fill | Заполняет ширину таблицы |
| RowHeadersVisible | False | Убирает служебную колонку |

## Шаг 6. Соберите ProductEditForm

Добавьте подписи и элементы:

| Тип | (Name) | Данные |
| --- | --- | --- |
| TextBox | `txtArticle` | Артикул |
| TextBox | `txtName` | Название |
| ComboBox | `cboCategory` | Категория |
| ComboBox | `cboManufacturer` | Производитель |
| ComboBox | `cboSupplier` | Поставщик |
| ComboBox | `cboUnit` | Единица измерения |
| NumericUpDown | `numPrice` | Цена |
| NumericUpDown | `numDiscount` | Скидка |
| NumericUpDown | `numStock` | Остаток |
| TextBox | `txtDescription` | Описание |
| TextBox | `txtImagePath` | Путь к изображению |
| Button | `btnBrowse` | Выбрать изображение |
| Button | `btnSave` | Сохранить |
| Button | `btnCancel` | Отмена |

Настройки числовых полей:

| Элемент | Maximum | DecimalPlaces |
| --- | ---: | ---: |
| `numPrice` | 1000000 | 2 |
| `numDiscount` | 100 | 2 |
| `numStock` | 1000000 | 0 |

У `txtDescription` установите `Multiline = True`.

## Шаг 7. Соберите OrdersForm

Добавьте:

| Тип | (Name) | Назначение |
| --- | --- | --- |
| DataGridView | `dgvOrders` | Список заказов |
| Panel | `pnlAdminActions` | Действия только администратора |
| ComboBox | `cboStatus` | Новый статус |
| Button | `btnSaveStatus` | Сохранить статус |
| Button | `btnDelete` | Удалить заказ |
| Button | `btnRefresh` | Обновить |
| Button | `btnClose` | Назад |

Для `dgvOrders` задайте те же свойства, что и для `dgvProducts`.

# Часть 3. Сеанс пользователя

## Шаг 8. Создайте Session.cs

Добавьте в проект обычный класс `Session.cs`:

```csharp
namespace ShoeStoreApp;

public static class Session
{
    public static int? UserId { get; set; }
    public static string FullName { get; set; } = "Гость";
    public static string Role { get; set; } = "guest";

    public static bool IsAdministrator => Role == "administrator";
    public static bool IsManager => Role == "manager";
    public static bool CanManageCatalog => IsAdministrator || IsManager;

    public static void StartGuest()
    {
        UserId = null;
        FullName = "Гость";
        Role = "guest";
    }
}
```

Этот класс хранит данные того, кто вошёл. `static` позволяет обратиться к ним из любой формы.

## Шаг 9. Создайте AuthResult

Создайте `Models.cs`:

```csharp
namespace ShoeStoreApp;

public sealed class AuthResult
{
    public int Id { get; init; }
    public string FullName { get; init; } = "";
    public string Role { get; init; } = "";
}
```

Объект `AuthResult` переносит из базы номер, ФИО и роль найденного пользователя.

# Часть 4. Подключение к MySQL

## Шаг 10. Создайте Database.cs

Создайте класс `Database.cs` и добавьте начало:

```csharp
using MySql.Data.MySqlClient;
using System.Data;

namespace ShoeStoreApp;

public static class Database
{
    private const string ConnectionString =
        "Server=localhost;Port=3306;Database=shoe_store;" +
        "Uid=root;Pwd=YOUR_PASSWORD;" +
        "SslMode=None;AllowPublicKeyRetrieval=True;";
}
```

Замените `YOUR_PASSWORD` на пароль MySQL. Используйте те же данные, с которыми подключаетесь через Workbench.

| Часть | Значение |
| --- | --- |
| Server | Адрес MySQL Server |
| Port | Порт, обычно 3306 |
| Database | Название базы |
| Uid | Пользователь MySQL |
| Pwd | Пароль MySQL |

## Шаг 11. Разберите четыре действия SQL

| Действие | SQL | Метод C# |
| --- | --- | --- |
| Получить данные | `SELECT` | `ExecuteReader`, `ExecuteScalar` или адаптер |
| Добавить строку | `INSERT` | `ExecuteNonQuery` |
| Изменить строку | `UPDATE` | `ExecuteNonQuery` |
| Удалить строку | `DELETE` | `ExecuteNonQuery` |

Обычный запрос выполняется так:

```csharp
using MySqlConnection connection = new(ConnectionString);
connection.Open();

const string sql = "SELECT id FROM users WHERE login = @login;";
using MySqlCommand command = new(sql, connection);
command.Parameters.AddWithValue("@login", login);
```

Параметр `@login` безопасно передаёт значение отдельно от SQL. Не собирайте запрос из текста пользователя через `+`.

## Шаг 12. Добавьте авторизацию

Внутрь `Database` добавьте:

```csharp
public static AuthResult? Authenticate(string login, string password)
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();

    const string sql = @"
        SELECT id,
               CONCAT_WS(' ', last_name, first_name, middle_name) AS full_name,
               role
        FROM users
        WHERE login = @login AND password = @password
        LIMIT 1;";

    using MySqlCommand command = new(sql, connection);
    command.Parameters.AddWithValue("@login", login);
    command.Parameters.AddWithValue("@password", password);

    using MySqlDataReader reader = command.ExecuteReader();
    if (!reader.Read()) return null;

    return new AuthResult
    {
        Id = reader.GetInt32("id"),
        FullName = reader.GetString("full_name"),
        Role = reader.GetString("role")
    };
}
```

`WHERE` ищет строку с подходящими логином и паролем. `LIMIT 1` говорит, что достаточно одной строки.

## Шаг 13. Добавьте загрузку таблицы

В конец `Database` добавьте вспомогательный метод:

```csharp
private static DataTable FillTable(MySqlCommand command)
{
    DataTable table = new();
    using MySqlDataAdapter adapter = new(command);
    adapter.Fill(table);
    return table;
}
```

Адаптер выполняет запрос и заполняет обычную `DataTable`. Затем её можно показать в `DataGridView`.

## Шаг 14. Добавьте SELECT товаров

```csharp
public static DataTable GetProducts(string search, int? supplierId, bool descending)
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();

    string direction = descending ? "DESC" : "ASC";
    string sql = @"
        SELECT p.article, p.name,
               c.name AS category,
               m.name AS manufacturer,
               s.name AS supplier,
               u.name AS unit,
               p.price, p.discount_percent,
               ROUND(p.price * (1 - p.discount_percent / 100), 2) AS final_price,
               p.stock_quantity, p.description, p.image_path
        FROM products p
        JOIN categories c ON c.id = p.category_id
        JOIN manufacturers m ON m.id = p.manufacturer_id
        JOIN suppliers s ON s.id = p.supplier_id
        JOIN units u ON u.id = p.unit_id
        WHERE (@search = ''
               OR p.article LIKE CONCAT('%', @search, '%')
               OR p.name LIKE CONCAT('%', @search, '%')
               OR p.description LIKE CONCAT('%', @search, '%')
               OR c.name LIKE CONCAT('%', @search, '%')
               OR m.name LIKE CONCAT('%', @search, '%')
               OR s.name LIKE CONCAT('%', @search, '%'))
          AND (@supplierId IS NULL OR p.supplier_id = @supplierId)
        ORDER BY p.stock_quantity " + direction + ", p.name;";

    using MySqlCommand command = new(sql, connection);
    command.Parameters.AddWithValue("@search", search);
    command.Parameters.AddWithValue("@supplierId",
        supplierId.HasValue ? supplierId.Value : DBNull.Value);
    return FillTable(command);
}
```

Здесь:

- `JOIN` получает названия из связанных таблиц;
- `LIKE` ищет часть текста;
- условие с `supplierId` включает фильтр;
- `ORDER BY` сортирует остаток;
- `ROUND` вычисляет цену со скидкой.

Направление сортировки выбирается только между двумя значениями нашего кода, поэтому пользователь не может подставить произвольный SQL.

## Шаг 15. Добавьте загрузку справочника

```csharp
public static DataTable GetLookup(string tableName)
{
    string[] allowed = { "categories", "manufacturers", "suppliers", "units" };
    if (!allowed.Contains(tableName))
        throw new ArgumentException("Недопустимый справочник.");

    using MySqlConnection connection = new(ConnectionString);
    connection.Open();
    using MySqlCommand command = new(
        $"SELECT id, name FROM {tableName} ORDER BY name;", connection);
    return FillTable(command);
}

public static DataTable GetSuppliersWithAll()
{
    DataTable table = GetLookup("suppliers");
    DataRow row = table.NewRow();
    row["id"] = 0;
    row["name"] = "Все поставщики";
    table.Rows.InsertAt(row, 0);
    return table;
}
```

Список `allowed` обязателен, потому что название таблицы нельзя передать обычным SQL-параметром.

# Часть 5. Авторизация

## Шаг 16. Напишите обработчик входа

Откройте `LoginForm` в конструкторе, дважды щёлкните `btnLogin` и добавьте:

```csharp
private void btnLogin_Click(object sender, EventArgs e)
{
    string login = txtLogin.Text.Trim();
    string password = txtPassword.Text;

    if (login == "" || password == "")
    {
        MessageBox.Show("Введите логин и пароль.");
        return;
    }

    try
    {
        AuthResult? user = Database.Authenticate(login, password);
        if (user == null)
        {
            MessageBox.Show("Неверный логин или пароль.");
            return;
        }

        Session.UserId = user.Id;
        Session.FullName = user.FullName;
        Session.Role = user.Role;
        OpenCatalog();
    }
    catch (Exception ex)
    {
        MessageBox.Show("Не удалось подключиться к базе.\n" + ex.Message);
    }
}
```

`Database.Authenticate` вызывается именно здесь — после проверки заполнения полей.

## Шаг 17. Добавьте гостевой вход и переход

```csharp
private void btnGuest_Click(object sender, EventArgs e)
{
    Session.StartGuest();
    OpenCatalog();
}

private void OpenCatalog()
{
    Hide();
    using ProductsForm form = new();
    form.ShowDialog();
    Show();
    txtPassword.Clear();
}
```

Подключите события:

```text
btnLogin → Click → btnLogin_Click
btnGuest → Click → btnGuest_Click
```

## Шаг 18. Измените запускаемую форму

Откройте `Program.cs` и убедитесь, что запускается:

```csharp
Application.Run(new LoginForm());
```

# Часть 6. Каталог товаров

## Шаг 19. Настройте права при загрузке

В `ProductsForm.cs`:

```csharp
private void ProductsForm_Load(object sender, EventArgs e)
{
    lblCurrentUser.Text = $"{Session.FullName} ({Session.Role})";
    pnlManagement.Visible = Session.CanManageCatalog;
    btnAdd.Visible = Session.IsAdministrator;
    btnEdit.Visible = Session.IsAdministrator;
    btnDelete.Visible = Session.IsAdministrator;
    btnOrders.Visible = Session.IsAdministrator || Session.IsManager;

    if (Session.CanManageCatalog)
    {
        cboSupplier.DataSource = Database.GetSuppliersWithAll();
        cboSupplier.DisplayMember = "name";
        cboSupplier.ValueMember = "id";
        cboSort.Items.AddRange(new object[]
        {
            "Остаток: по возрастанию",
            "Остаток: по убыванию"
        });
        cboSort.SelectedIndex = 0;
    }

    LoadProducts();
}
```

## Шаг 20. Загружайте товары

```csharp
private void LoadProducts()
{
    string search = Session.CanManageCatalog ? txtSearch.Text.Trim() : "";
    int? supplierId = null;

    if (Session.CanManageCatalog && cboSupplier.SelectedValue != null &&
        Convert.ToInt32(cboSupplier.SelectedValue) != 0)
        supplierId = Convert.ToInt32(cboSupplier.SelectedValue);

    bool descending = Session.CanManageCatalog && cboSort.SelectedIndex == 1;
    dgvProducts.DataSource = Database.GetProducts(search, supplierId, descending);
}
```

Этот метод вызывается:

- при загрузке формы;
- после изменения поиска;
- после выбора поставщика;
- после смены сортировки;
- после добавления, изменения или удаления товара.

Подключите:

```text
ProductsForm → Load → ProductsForm_Load
txtSearch → TextChanged → txtSearch_TextChanged
cboSupplier → SelectedIndexChanged → cboSupplier_SelectedIndexChanged
cboSort → SelectedIndexChanged → cboSort_SelectedIndexChanged
```

Обработчики:

```csharp
private void txtSearch_TextChanged(object sender, EventArgs e) => LoadProducts();
private void cboSupplier_SelectedIndexChanged(object sender, EventArgs e) => LoadProducts();
private void cboSort_SelectedIndexChanged(object sender, EventArgs e) => LoadProducts();
```

## Шаг 21. Откройте форму товара

```csharp
private string? GetSelectedArticle()
{
    return dgvProducts.CurrentRow?.Cells["article"].Value?.ToString();
}

private void btnAdd_Click(object sender, EventArgs e)
{
    using ProductEditForm form = new(null);
    if (form.ShowDialog() == DialogResult.OK) LoadProducts();
}

private void btnEdit_Click(object sender, EventArgs e)
{
    string? article = GetSelectedArticle();
    if (article == null) { MessageBox.Show("Выберите товар."); return; }

    using ProductEditForm form = new(article);
    if (form.ShowDialog() == DialogResult.OK) LoadProducts();
}
```

`null` означает добавление, а существующий артикул — редактирование.

# Часть 7. INSERT, UPDATE и DELETE товара

## Шаг 22. Создайте ProductInput

В `Models.cs` добавьте класс со свойствами, совпадающими с полями формы:

```csharp
public sealed class ProductInput
{
    public string Article { get; init; } = "";
    public string Name { get; init; } = "";
    public int CategoryId { get; init; }
    public int ManufacturerId { get; init; }
    public int SupplierId { get; init; }
    public int UnitId { get; init; }
    public decimal Price { get; init; }
    public decimal DiscountPercent { get; init; }
    public int StockQuantity { get; init; }
    public string? Description { get; init; }
    public string? ImagePath { get; init; }
}
```

## Шаг 23. Добавьте INSERT

В `Database` создайте `AddProduct`. Запрос должен иметь вид:

```csharp
const string sql = @"
    INSERT INTO products
        (article, name, category_id, manufacturer_id, supplier_id,
         unit_id, price, discount_percent, stock_quantity,
         description, image_path)
    VALUES
        (@article, @name, @categoryId, @manufacturerId, @supplierId,
         @unitId, @price, @discount, @stock, @description, @imagePath);";
```

Создайте соединение и команду так же, как раньше. Для каждого `@параметра` вызовите `command.Parameters.AddWithValue`, затем:

```csharp
command.ExecuteNonQuery();
```

## Шаг 24. Добавьте UPDATE

Метод `UpdateProduct` получает старый артикул и новые данные. Основной запрос:

```csharp
const string sql = @"
    UPDATE products
    SET article = @article, name = @name,
        category_id = @categoryId, manufacturer_id = @manufacturerId,
        supplier_id = @supplierId, unit_id = @unitId,
        price = @price, discount_percent = @discount,
        stock_quantity = @stock, description = @description,
        image_path = @imagePath
    WHERE article = @oldArticle;";
```

`WHERE` обязателен. Без него изменятся все товары.

## Шаг 25. Добавьте DELETE с проверкой

```csharp
public static bool ProductIsUsedInOrders(string article)
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();
    using MySqlCommand command = new(
        "SELECT COUNT(*) FROM order_items WHERE product_article = @article;",
        connection);
    command.Parameters.AddWithValue("@article", article);
    return Convert.ToInt32(command.ExecuteScalar()) > 0;
}

public static void DeleteProduct(string article)
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();
    using MySqlCommand command = new(
        "DELETE FROM products WHERE article = @article;", connection);
    command.Parameters.AddWithValue("@article", article);
    command.ExecuteNonQuery();
}
```

В `btnDelete_Click` сначала вызовите `ProductIsUsedInOrders`. Если метод вернул `true`, покажите сообщение и не вызывайте `DeleteProduct`. Перед удалением обязательно спросите подтверждение через `MessageBoxButtons.YesNo`.

## Шаг 26. Заполните ProductEditForm

В событии `Load` привяжите справочники:

```csharp
private void BindLookup(ComboBox box, string table)
{
    box.DataSource = Database.GetLookup(table);
    box.DisplayMember = "name";
    box.ValueMember = "id";
}
```

Вызовите его для четырёх списков. В `btnSave_Click` проверьте артикул и название, соберите `ProductInput`, затем вызовите:

```csharp
if (originalArticle == null)
    Database.AddProduct(product);
else
    Database.UpdateProduct(originalArticle, product);

DialogResult = DialogResult.OK;
Close();
```

Оберните сохранение в `try/catch`, чтобы ошибка базы не закрыла приложение.

# Часть 8. Заказы

## Шаг 27. Добавьте SELECT заказов

В `Database.GetOrders` используйте запрос:

```sql
SELECT o.id, o.order_number,
       CONCAT_WS(' ', u.last_name, u.first_name, u.middle_name) AS customer,
       pp.address AS pickup_point,
       o.order_date, o.delivery_date, o.pickup_code, o.status,
       GROUP_CONCAT(CONCAT(oi.product_article, ' × ', oi.quantity)
                    SEPARATOR ', ') AS items
FROM customer_orders o
JOIN users u ON u.id = o.customer_id
JOIN pickup_points pp ON pp.id = o.pickup_point_id
JOIN order_items oi ON oi.order_id = o.id
GROUP BY o.id, o.order_number, customer, pickup_point,
         o.order_date, o.delivery_date, o.pickup_code, o.status
ORDER BY o.order_date DESC;
```

Выполните его через `MySqlCommand` и `FillTable`.

`GROUP_CONCAT` собирает несколько строк состава заказа в одну понятную строку только для показа. В самой базе товары по-прежнему хранятся раздельно.

## Шаг 28. Загрузите OrdersForm

```csharp
private void OrdersForm_Load(object sender, EventArgs e)
{
    cboStatus.Items.AddRange(
        new object[] { "new", "completed", "cancelled" });
    cboStatus.SelectedIndex = 0;
    pnlAdminActions.Visible = Session.IsAdministrator;
    LoadOrders();
}

private void LoadOrders()
{
    dgvOrders.DataSource = Database.GetOrders();
    dgvOrders.Columns["id"].Visible = false;
}
```

Менеджер видит заказы, но панель изменения скрыта. Администратор видит её.

## Шаг 29. Измените статус

В `Database`:

```csharp
public static void UpdateOrderStatus(int id, string status)
{
    string[] allowed = { "new", "completed", "cancelled" };
    if (!allowed.Contains(status))
        throw new ArgumentException("Недопустимый статус.");

    using MySqlConnection connection = new(ConnectionString);
    connection.Open();
    using MySqlCommand command = new(
        "UPDATE customer_orders SET status = @status WHERE id = @id;",
        connection);
    command.Parameters.AddWithValue("@status", status);
    command.Parameters.AddWithValue("@id", id);
    command.ExecuteNonQuery();
}
```

Вызывайте метод в `btnSaveStatus_Click`, передав `id` выбранной строки и `cboStatus.Text`. После изменения вызовите `LoadOrders()`.

## Шаг 30. Удалите заказ через транзакцию

Сначала удаляются строки `order_items`, затем сам заказ. Обе команды должны выполниться вместе:

```csharp
using MySqlConnection connection = new(ConnectionString);
connection.Open();
using MySqlTransaction transaction = connection.BeginTransaction();

try
{
    // DELETE FROM order_items WHERE order_id = @id
    // DELETE FROM customer_orders WHERE id = @id
    transaction.Commit();
}
catch
{
    transaction.Rollback();
    throw;
}
```

`Commit` подтверждает обе команды. `Rollback` отменяет изменения, если одна команда завершилась ошибкой.

# Часть 9. События

## Шаг 31. Проверьте все события

Откройте свойства элемента, нажмите значок молнии и проверьте:

| Элемент | Событие | Метод |
| --- | --- | --- |
| `LoginForm` | — | запускается из Program.cs |
| `btnLogin` | Click | `btnLogin_Click` |
| `btnGuest` | Click | `btnGuest_Click` |
| `ProductsForm` | Load | `ProductsForm_Load` |
| `txtSearch` | TextChanged | `txtSearch_TextChanged` |
| `cboSupplier` | SelectedIndexChanged | `cboSupplier_SelectedIndexChanged` |
| `cboSort` | SelectedIndexChanged | `cboSort_SelectedIndexChanged` |
| `btnAdd` | Click | `btnAdd_Click` |
| `btnEdit` | Click | `btnEdit_Click` |
| `btnDelete` | Click | `btnDelete_Click` |
| `btnOrders` | Click | `btnOrders_Click` |
| `ProductEditForm` | Load | `ProductEditForm_Load` |
| `btnSave` | Click | `btnSave_Click` |
| `btnCancel` | Click | `btnCancel_Click` |
| `OrdersForm` | Load | `OrdersForm_Load` |
| `btnSaveStatus` | Click | `btnSaveStatus_Click` |
| `btnDelete` | Click | `btnDelete_Click` |

Если метод написан, но событие не подключено, Visual Studio его не вызовет.

# Часть 10. Проверка

## Шаг 32. Проверьте подключение

1. Убедитесь, что MySQL Server запущен.
2. Проверьте базу `shoe_store` через Workbench.
3. Проверьте пароль в `ConnectionString`.
4. Соберите проект через **Сборка → Собрать решение**.
5. Запустите приложение.

## Шаг 33. Проверьте роли

| Роль | Логин | Пароль | Ожидаемый результат |
| --- | --- | --- | --- |
| Администратор | admin | admin | Все кнопки доступны |
| Менеджер | manager | manager | Поиск, фильтр, сортировка и заказы |
| Клиент | client | client | Только просмотр каталога |
| Гость | кнопка гостя | — | Только просмотр каталога |

## Шаг 34. Проверьте запросы

1. Найдите товар по части названия.
2. Выберите одного поставщика.
3. Измените сортировку остатка.
4. Добавьте новый товар.
5. Измените его цену.
6. Удалите товар, которого нет в заказах.
7. Попробуйте удалить товар из заказа — программа должна запретить действие.
8. Откройте заказы как менеджер.
9. Измените статус заказа как администратор.

# Частые ошибки

## Unable to connect

Проверьте сервер, порт, логин и пароль в `ConnectionString`. Они должны совпадать с подключением Workbench.

## Unknown database shoe_store

База не создана или в названии допущена ошибка.

## MySqlConnection не найден

Пакет `MySql.Data` не установлен либо отсутствует:

```csharp
using MySql.Data.MySqlClient;
```

## Кнопка ничего не делает

Проверьте событие `Click` через значок молнии в окне свойств.

## DataGridView пустой

Проверьте данные через Workbench. Затем поставьте точку останова в `LoadProducts` и убедитесь, что метод вызывается.

## Column not found

Имя колонки в C# должно совпадать с псевдонимом после `AS` в SQL.

# Что нужно отправить

- папку исходного проекта Windows Forms;
- собранную программу;
- ссылку на GitHub-репозиторий проекта.

Инструкция по первой отправке: [Как загрузить проект на GitHub](https://github.com/pgk-lectures/github-first-push).

Эта работа закрывает авторизацию, каталог, CRUD товаров и основную работу со списком заказов. Добавление заказа с произвольным набором товаров можно выполнить отдельным следующим этапом после освоения связей и транзакций.
