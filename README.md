    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using System.Xml.Linq;

    // Класс сотрудника
    public class Employee
    {
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public List<string> Positions { get; set; } = new List<string>(); // Список должностей сотрудника
    public decimal Salary { get; set; }

    public override string ToString()
    {
        return $"ID: {Id}, Имя: {FirstName} {LastName}, Должности: {string.Join(", ", Positions)}, Зарплата: {Salary}";
    }
    }

    // Класс описания должности
    public class JobDescription
    {
    public string Title { get; set; }               // Название должности
    public int VacanciesCount { get; set; }         // Доступные вакансии
    public decimal BaseSalary { get; set; }         // Размер зарплаты по должности
    }

    // Класс управления персоналом (Отдел кадров)
    public class PersonnelDepartment
    {
    private List<Employee> employees = new List<Employee>();                  // Хранение сотрудников
    private Dictionary<string, JobDescription> jobDescriptions = new Dictionary<string, JobDescription>(); // Хранение вакансий

    // Добавление вакансии в штатное расписание
    public void AddJobDescription(string title, int vacanciesCount, decimal baseSalary)
    {
        jobDescriptions[title] = new JobDescription
        {
            Title = title,
            VacanciesCount = vacanciesCount,
            BaseSalary = baseSalary
        };
    }

    // Принять сотрудника на работу
    public void HireEmployee(int id, string firstName, string lastName, string position)
    {
        // Проверка наличия вакансии
        if (!jobDescriptions.ContainsKey(position) || jobDescriptions[position].VacanciesCount <= 0)
        {
            throw new Exception("Свободных вакансий нет.");
        }

        // Создание нового сотрудника
        var employee = new Employee
        {
            Id = id,
            FirstName = firstName,
            LastName = lastName,
            Positions = new List<string>() { position }, // Первоначальная должность
            Salary = jobDescriptions[position].BaseSalary
        };

        employees.Add(employee);
        jobDescriptions[position].VacanciesCount--; // Уменьшаем количество вакансий
    }

    // Совместительство: назначить дополнительную должность сотруднику
    public void AddAdditionalPosition(int employeeId, string additionalPosition)
    {
        var employee = employees.FirstOrDefault(e => e.Id == employeeId);
        if (employee == null)
        {
            throw new Exception("Сотрудник не найден.");
        }

        if (!jobDescriptions.ContainsKey(additionalPosition) || jobDescriptions[additionalPosition].VacanciesCount <= 0)
        {
            throw new Exception("Нет свободных вакансий на запрашиваемую должность.");
        }

        employee.Positions.Add(additionalPosition);
        employee.Salary += jobDescriptions[additionalPosition].BaseSalary;
        jobDescriptions[additionalPosition].VacanciesCount--;
    }

    // Увольнить сотрудника
    public void DismissEmployee(int employeeId)
    {
        var employee = employees.FirstOrDefault(e => e.Id == employeeId);
        if (employee == null)
        {
            throw new Exception("Сотрудник не найден.");
        }

        foreach (var pos in employee.Positions)
        {
            jobDescriptions[pos].VacanciesCount++; // Восстанавливаем вакансии
        }

        employees.Remove(employee);
    }

    // Вывод сотрудников в виде таблицы
    public void PrintEmployeesTable()
    {
        const int columnWidth = 20; // Ширина столбца
        string separator = "+--------------------+--------------------+--------------------+--------------------+\n"; // Граница таблицы

        Console.Write(separator);
        Console.Write("|{0,-20}|{1,-20}|{2,-20}|{3,-20}|\n", "ID", "ФИО", "Должности", "Зарплата");
        Console.Write(separator);

        foreach (var employee in employees)
        {
            Console.Write("|{0,-20}|{1,-20}|{2,-20}|{3,-20}|\n",
                employee.Id,
                employee.FirstName + " " + employee.LastName,
                string.Join(", ", employee.Positions),
                employee.Salary.ToString("C")); // Конвертирование зарплаты в денежный формат
        }

        Console.Write(separator);
    }

    // Сохранение сотрудников в XML-файл
    public void SaveToXml(string filePath)
    {
        var root = new XElement("Employees",
            employees.Select(e => new XElement("Employee",
                new XElement("Id", e.Id),
                new XElement("FirstName", e.FirstName),
                new XElement("LastName", e.LastName),
                new XElement("Positions", string.Join(",", e.Positions)),
                new XElement("Salary", e.Salary))));

        root.Save(filePath);
    }

    // Импорт сотрудников из XML-файла
    public void LoadFromXml(string filePath)
    {
        if (File.Exists(filePath))
        {
            var doc = XElement.Load(filePath);
            employees.Clear();
            foreach (var employeeElem in doc.Elements("Employee"))
            {
                var id = int.Parse(employeeElem.Element("Id").Value);
                var firstName = employeeElem.Element("FirstName").Value;
                var lastName = employeeElem.Element("LastName").Value;
                var positions = employeeElem.Element("Positions").Value.Split(',');
                var salary = decimal.Parse(employeeElem.Element("Salary").Value);

                var employee = new Employee
                {
                    Id = id,
                    FirstName = firstName,
                    LastName = lastName,
                    Positions = positions.Select(p => p.Trim()).ToList(),
                    Salary = salary
                };

                employees.Add(employee);
            }
        }
    }

    // Получение сотрудников по критерию
    public IEnumerable<Employee> FilterEmployees(Func<Employee, bool> predicate)
    {
        return employees.Where(predicate);
    }

    // Сортировка сотрудников по зарплате
    public IEnumerable<Employee> SortEmployeesBySalary(bool ascending = true)
    {
        return ascending ? employees.OrderBy(e => e.Salary) : employees.OrderByDescending(e => e.Salary);
    }
    }

    // Точка входа в программу
    class Program
    {
    static void Main()
    {
        var personnel = new PersonnelDepartment();

        // Добавляем вакансии в штатное расписание
        personnel.AddJobDescription("Генеральный директор", 1, 150000m);
        personnel.AddJobDescription("Начальник отдела", 3, 100000m);
        personnel.AddJobDescription("Программист", 5, 80000m);
        personnel.AddJobDescription("Бухгалтер", 2, 60000m);

        // Прием сотрудников на работу
        personnel.HireEmployee(1, "Иванов", "Иван Иванович", "Генеральный директор");
        personnel.HireEmployee(2, "Семёнова", "Марина Олеговна", "Начальник отдела");
        personnel.HireEmployee(3, "Петров", "Алексей Сергеевич", "Программист");
        personnel.HireEmployee(4, "Николаева", "Ольга Александровна", "Бухгалтер");

        // Назначение сотруднику второй должности (совмещение)
        personnel.AddAdditionalPosition(3, "Начальник отдела");

        // Выводим сотрудников в виде таблицы
        personnel.PrintEmployeesTable();

        // Сохраняем сотрудников в XML-файл
        personnel.SaveToXml("employees.xml");

        // Читаем сотрудников из XML-файла
        personnel.LoadFromXml("employees.xml");

        // Фильтруем сотрудников по определенным критериям
        var filteredEmployees = personnel.FilterEmployees(e => e.FirstName.Contains("Иван") || e.LastName.Contains("Алекс"));
        Console.WriteLine("\nФильтрованные сотрудники:");
        foreach (var emp in filteredEmployees)
        {
            Console.WriteLine(emp);
        }
        // Сортируем сотрудников по зарплате (убывание)
        var sortedEmployees = personnel

        .SortEmployeesBySalary(false);
        Console.WriteLine("\nСотрудники, отсортированные по зарплате (убывание):");
        foreach (var emp in sortedEmployees)
        {
            Console.WriteLine(emp);
        }

        // Увольняем сотрудника
        personnel.DismissEmployee(2);

        // Повторно выводим сотрудников после увольнения
        personnel.PrintEmployeesTable();
    }
    }