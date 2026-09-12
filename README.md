# Windows Forms и MySQL: как самостоятельно написать приложение

Эта лекция объясняет инструменты, необходимые для программы ДЭ. Готовых форм, таблиц ООО «Обувь» и запросов к ним здесь нет: интерфейс и логику нужно разработать самостоятельно по своей базе.

Примеры используют нейтральные `example_groups` и `example_items` и не являются решением задания.

## Что вы изучите

- подключение к MySQL;
- `SELECT`, `INSERT`, `UPDATE`, `DELETE`;
- параметры запросов;
- `DataGridView` и `ComboBox`;
- поиск, фильтрацию и сортировку;
- авторизацию и роли;
- `NULL`, числа и даты;
- транзакции и обработку ошибок.

# 1. Подготовка проекта

1. Создайте **Windows Forms App** на C# и .NET 8.
2. Запустите пустую форму.
3. Через **Управление пакетами NuGet** установите `MySql.Data` от Oracle.
4. Создавайте интерфейс мышкой через Designer.

Полезные элементы: `Label`, `TextBox`, `Button`, `ComboBox`, `NumericUpDown`, `DateTimePicker`, `PictureBox`, `DataGridView`.

# 2. Класс Database

SQL лучше хранить отдельно от форм. Создайте `Database.cs`:

```csharp
using MySql.Data.MySqlClient;
using System.Data;

namespace YourProject;

public static class Database
{
    private const string ConnectionString =
        "Server=localhost;Port=3306;Database=YOUR_DATABASE;" +
        "Uid=root;Pwd=YOUR_PASSWORD;" +
        "SslMode=None;AllowPublicKeyRetrieval=True;";
}
```

Используйте те же сервер, порт, пользователя и пароль, что в Workbench. Замените `YOUR_DATABASE` и `YOUR_PASSWORD`.

# 3. Выполнение запроса

```csharp
using MySqlConnection connection = new(ConnectionString);
connection.Open();

const string sql = "SELECT name FROM example_items WHERE id = @id;";
using MySqlCommand command = new(sql, connection);
command.Parameters.AddWithValue("@id", itemId);
```

`using` закроет соединение. `@id` передаётся отдельно от SQL.

Никогда не собирайте SQL из текста пользователя через `+`: это ломается на кавычках и допускает SQL-инъекцию.

# 4. Четыре способа выполнения

| Метод | Когда использовать |
| --- | --- |
| `ExecuteScalar()` | Получить одно значение |
| `ExecuteReader()` | Прочитать строки вручную |
| `MySqlDataAdapter` | Получить `DataTable` |
| `ExecuteNonQuery()` | Выполнить `INSERT`, `UPDATE`, `DELETE` |

## Одно значение

```csharp
int count = Convert.ToInt32(command.ExecuteScalar());
```

## Одна найденная строка

```csharp
using MySqlDataReader reader = command.ExecuteReader();
if (reader.Read())
{
    int id = reader.GetInt32("id");
    string name = reader.GetString("name");
}
```

## Таблица

```csharp
DataTable table = new();
using MySqlDataAdapter adapter = new(command);
adapter.Fill(table);
return table;
```

# 5. SELECT и JOIN

```csharp
public static DataTable GetItems()
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();

    const string sql = @"
        SELECT i.id, i.name,
               g.name AS group_name,
               i.amount
        FROM example_items i
        JOIN example_groups g ON g.id = i.group_id
        ORDER BY i.name;";

    using MySqlCommand command = new(sql, connection);
    DataTable table = new();
    using MySqlDataAdapter adapter = new(command);
    adapter.Fill(table);
    return table;
}
```

`JOIN` заменяет внешний ключ понятным названием связанной записи. Составьте собственные `JOIN` по своей ER-диаграмме.

# 6. DataGridView

На форме:

```csharp
private void LoadItems()
{
    dgvItems.DataSource = Database.GetItems();
    dgvItems.Columns["id"].Visible = false;
    dgvItems.Columns["name"].HeaderText = "Название";
    dgvItems.Columns["group_name"].HeaderText = "Группа";
}

private void MainForm_Load(object sender, EventArgs e)
{
    LoadItems();
}
```

Подключите событие `Form.Load` через значок молнии в свойствах.

Настройки `DataGridView`:

| Свойство | Значение |
| --- | --- |
| ReadOnly | True |
| AllowUserToAddRows | False |
| MultiSelect | False |
| SelectionMode | FullRowSelect |
| AutoSizeColumnsMode | Fill |
| RowHeadersVisible | False |

Получение скрытого ключа выбранной строки:

```csharp
int id = Convert.ToInt32(dgvItems.CurrentRow.Cells["id"].Value);
```

# 7. ComboBox из справочника

```csharp
public static DataTable GetGroups()
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();
    using MySqlCommand command = new(
        "SELECT id, name FROM example_groups ORDER BY name;", connection);

    DataTable table = new();
    using MySqlDataAdapter adapter = new(command);
    adapter.Fill(table);
    return table;
}
```

```csharp
cboGroup.DataSource = Database.GetGroups();
cboGroup.DisplayMember = "name";
cboGroup.ValueMember = "id";
```

Пользователь видит название, а программа получает ключ:

```csharp
int groupId = Convert.ToInt32(cboGroup.SelectedValue);
```

# 8. Поиск через LIKE

```sql
WHERE name LIKE CONCAT('%', @search, '%')
```

```csharp
command.Parameters.AddWithValue("@search", search);
```

Вызывайте повторную загрузку из `TextChanged`:

```csharp
private void txtSearch_TextChanged(object sender, EventArgs e)
{
    LoadItems();
}
```

# 9. Фильтрация

```sql
WHERE (@group_id IS NULL OR group_id = @group_id)
```

```csharp
command.Parameters.AddWithValue(
    "@group_id",
    groupId.HasValue ? groupId.Value : DBNull.Value);
```

`NULL` выключает фильтр. Пункт «Все» можно добавить только в `DataTable` списка, не сохраняя его в базе.

# 10. Сортировка

```csharp
string direction = descending ? "DESC" : "ASC";
string sql = "SELECT * FROM example_items ORDER BY amount " + direction;
```

`ASC` и `DESC` выбираются только кодом. Не подставляйте произвольный текст пользователя.

# 11. INSERT

```csharp
public static void AddItem(string name, int groupId, int amount)
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();
    const string sql = @"
        INSERT INTO example_items (name, group_id, amount)
        VALUES (@name, @group_id, @amount);";

    using MySqlCommand command = new(sql, connection);
    command.Parameters.AddWithValue("@name", name);
    command.Parameters.AddWithValue("@group_id", groupId);
    command.Parameters.AddWithValue("@amount", amount);
    command.ExecuteNonQuery();
}
```

Вызывайте метод в `btnSave_Click` после проверки полей. После успеха обновите основную таблицу.

# 12. UPDATE

```csharp
public static void UpdateItem(int id, string name, int groupId, int amount)
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();
    const string sql = @"
        UPDATE example_items
        SET name = @name, group_id = @group_id, amount = @amount
        WHERE id = @id;";

    using MySqlCommand command = new(sql, connection);
    command.Parameters.AddWithValue("@id", id);
    command.Parameters.AddWithValue("@name", name);
    command.Parameters.AddWithValue("@group_id", groupId);
    command.Parameters.AddWithValue("@amount", amount);
    command.ExecuteNonQuery();
}
```

Без `WHERE` будут изменены все строки.

# 13. DELETE

```csharp
public static void DeleteItem(int id)
{
    using MySqlConnection connection = new(ConnectionString);
    connection.Open();
    using MySqlCommand command = new(
        "DELETE FROM example_items WHERE id = @id;", connection);
    command.Parameters.AddWithValue("@id", id);
    command.ExecuteNonQuery();
}
```

Перед удалением проверьте выбранную строку, связанные данные и спросите подтверждение:

```csharp
DialogResult answer = MessageBox.Show(
    "Удалить выбранную запись?", "Подтверждение",
    MessageBoxButtons.YesNo, MessageBoxIcon.Warning);

if (answer != DialogResult.Yes) return;
```

# 14. Авторизация и Session

Общий алгоритм:

```text
Проверить заполнение полей
→ выполнить SELECT с логином и паролем
→ если reader.Read() == false, показать ошибку
→ сохранить id, ФИО и роль
→ открыть основную форму
```

```csharp
public static class Session
{
    public static int? UserId { get; set; }
    public static string FullName { get; set; } = "Гость";
    public static string Role { get; set; } = "guest";
}
```

SQL авторизации напишите по полям собственной базы. После входа ограничьте интерфейс:

```csharp
btnAdd.Visible = Session.Role == "administrator";
btnDelete.Visible = Session.Role == "administrator";
```

Скрытия кнопки недостаточно: важные ограничения должны проверяться логикой и базой.

# 15. ENUM, NULL, числа и даты

MySQL возвращает `ENUM` строкой:

```csharp
string status = reader.GetString("status");
```

Перед обновлением проверяйте значение по разрешённому списку.

Чтение `NULL`:

```csharp
string text = row["description"] == DBNull.Value
    ? "" : row["description"].ToString()!;
```

Передача `NULL`:

```csharp
command.Parameters.AddWithValue("@description",
    string.IsNullOrWhiteSpace(text) ? DBNull.Value : text);
```

Для чисел используйте `NumericUpDown`:

```csharp
decimal price = numPrice.Value;
int amount = Convert.ToInt32(numAmount.Value);
```

Для даты — `DateTimePicker`:

```csharp
command.Parameters.AddWithValue("@date", dtpDate.Value.Date);
```

# 16. Транзакции

Транзакция нужна, когда одно действие изменяет несколько связанных таблиц.

```csharp
using MySqlConnection connection = new(ConnectionString);
connection.Open();
using MySqlTransaction transaction = connection.BeginTransaction();

try
{
    // Выполните связанные команды с connection и transaction.
    transaction.Commit();
}
catch
{
    transaction.Rollback();
    throw;
}
```

После вставки строки с `AI` новый ключ доступен так:

```csharp
command.ExecuteNonQuery();
int newId = Convert.ToInt32(command.LastInsertedId);
```

Его можно использовать для связанных строк внутри той же транзакции.

# 17. Обработка ошибок

```csharp
try
{
    Database.AddItem(name, groupId, amount);
    MessageBox.Show("Данные сохранены.");
}
catch (MySqlException ex)
{
    MessageBox.Show("Не удалось сохранить данные.\n" + ex.Message,
        "Ошибка базы данных", MessageBoxButtons.OK, MessageBoxIcon.Error);
}
```

Приложение не должно аварийно закрываться.

# 18. Где вызывать методы

| Действие | Событие формы |
| --- | --- |
| Авторизация | `btnLogin_Click` |
| Первая загрузка | `Form.Load` |
| Поиск | `txtSearch_TextChanged` |
| Фильтр | `cboFilter_SelectedIndexChanged` |
| Добавление | `btnSave_Click` формы добавления |
| Изменение | `btnSave_Click` формы редактирования |
| Удаление | `btnDelete_Click` после подтверждения |
| Обновление | после успешного изменения базы |

Если метод написан, но событие не подключено через значок молнии, он не выполнится.

# 19. Самостоятельная работа

1. Определите формы из требований и ролей.
2. Назовите элементы управления понятно.
3. Напишите собственные `SELECT` и `JOIN` по своей ER-диаграмме.
4. Заполните справочники в `ComboBox`.
5. Реализуйте поиск, фильтрацию и сортировку.
6. Реализуйте разрешённые ролям `INSERT`, `UPDATE`, `DELETE`.
7. Для составных операций используйте транзакцию.
8. После изменений обновляйте интерфейс.
9. Проверьте неверный ввод и отключённый сервер.

# Самопроверка

- SQL находится в отдельном классе.
- Значения пользователя передаются параметрами.
- `DataGridView` показывает понятные связанные названия.
- `ComboBox.ValueMember` хранит ключ.
- Роли ограничивают действия.
- После изменений данные обновляются.
- Составные операции используют транзакцию.
- Перед удалением есть подтверждение.
- Ошибки не закрывают приложение.

# Частые ошибки

- `Unable to connect` — проверьте данные Workbench и строку подключения.
- `Unknown database` — неверно указана схема.
- `Table does not exist` — название не совпадает с вашей моделью.
- `MySqlConnection` не найден — не установлен `MySql.Data`.
- Пустой `DataGridView` — нет данных или не подключено событие `Load`.
- `Cannot add child row` — передан несуществующий внешний ключ.
- `Cannot delete parent row` — запись используется в другой таблице.

Инструкция по отправке: [Как загрузить проект на GitVerse](https://github.com/pgk-lectures/gitverse-first-push).

Готового решения программы в этой публичной репозитории нет. Названия форм, запросы и бизнес-логику нужно построить по собственному проекту базы и условию ДЭ.
