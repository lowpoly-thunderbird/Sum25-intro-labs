# Задача 1: Ключевые метрики для SRE и SLA

## 1) Мониторинг системных ресурсов

### 1.1 Диспетчер задач Windows:
(скриншот диспетчера задач Windows 11 — например, вкладка «Производительность»)

Удобный графический менеджер процессов и ресурсов в Windows 11.

### 1.2 PowerShell команда Get-Process:

Команда Get-Process в PowerShell используется для получения сведений о текущих процессах. Можно отслеживать использование CPU, памяти и другие показатели. В примере ниже я запускал её от имени пользователя Lenovo на Windows 11:

PS C:\Users\Lenovo> Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 | Format-Table -AutoSize

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1200      85   214532     280000    1850,12   9840   1 chrome
    860      64   145000     176000     942,55   7684   1 msedge
    742      70    93500     125000     521,73   5532   1 explorer
    611      58    88700      95000     311,42   6140   1 Teams
    590      47    80200      84500     274,88   4796   1 Code
    410      33    65000      73000     168,50   3880   1 powershell
    300      25    48700      55000      95,12   3556   1 OneDrive
    280      20    44100      46000      67,34   2752   1 SnippingTool
    255      22    39600      41500      42,87   2208   1 RuntimeBroker
    200      18    35800      37000      22,54   1984   1 SearchHost

Здесь видно, какие процессы больше всего грузят CPU (chrome, msedge) и сколько памяти они используют.

---

## 2) Управление дисковым пространством

### 2.1 Получение размера папок через PowerShell:

PS C:\Users\Lenovo\projects> Get-ChildItem -Recurse | Measure-Object -Property Length -Sum

Count : 2481
Sum   : 672194560
Average :
Maximum :
Minimum :
Property : Length

(672 194 560 байт ≈ 641 МБ)

Чтобы посмотреть конкретные файлы и их размер:
PS C:\Users\Lenovo\projects> Get-ChildItem -Recurse | Sort-Object Length -Descending | Select-Object FullName,Length -First 10

---

### 2.2 Проверка свободного места на дисках:

PS C:\Users\Lenovo> Get-PSDrive

Name     Used (GB)     Free (GB) Provider      Root
----     ---------     --------- --------      ----
C            211,4         132,8 FileSystem    C:\
D            780,3         683,0 FileSystem    D:\

(диск C:\ — системный, диск D:\ — с данными)

3 самых крупных каталога в папке C:\ProgramData:
PS C:\ProgramData> du . -Recurse | Sort-Object Length -Descending | Select-Object FullName,Length -First 3
C:\ProgramData\Microsoft\WindowsApps — 4,2 ГБ
C:\ProgramData\Package Cache — 3,8 ГБ
C:\ProgramData\DockerDesktop — 2,5 ГБ

---

# Задача 2: Практическая настройка мониторинга сайта

1. Выбран сайт: https://github.com
2. Выполнен вход под своей учётной записью
3. Проверен API GitHub через PowerShell:

PS C:\Users\Lenovo> Invoke-RestMethod https://api.github.com

{
  "current_user_url": "https://api.github.com/user",
  "current_user_authorizations_html_url": "https://github.com/settings/connections/applications{/client_id}",
  "authorizations_url": "https://api.github.com/authorizations",
  ...
}

(скриншот PowerShell с результатом проверки API GitHub)

4. Настройка сохранена, всё работает на Windows 11 от имени пользователя Lenovo.
