Aqui está o **código completo correto e atualizado** para o seu laboratório com:

* `QLabel`
* `QLineEdit`
* `QSpinBox`
* `QPushButton`
* `QMessageBox`

Exatamente como normalmente os professores pedem. 👨‍💻

---

# 1️⃣ `main.cpp`

Arquivo que inicia a aplicação.

```cpp
#include "mainwindow.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow w;
    w.show();
    return a.exec();
}
```

---

# 2️⃣ `mainwindow.h`

Define a classe da janela principal.

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>

QT_BEGIN_NAMESPACE
namespace Ui {
class MainWindow;
}
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private slots:
    void on_sendButton_clicked();

private:
    Ui::MainWindow *ui;
};

#endif // MAINWINDOW_H
```

---

# 3️⃣ `mainwindow.cpp`

Aqui está a lógica do programa.

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QMessageBox>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent),
      ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_sendButton_clicked()
{
    QString name = ui->nameInput->text();
    int age = ui->ageInput->value();

    QString message = "Hello " + name +
                      "\nYour age is: " +
                      QString::number(age);

    QMessageBox::information(this, "User Info", message);
}
```

---

# 4️⃣ Configuração da interface (`mainwindow.ui`)

No **Qt Designer**, configure assim:

### Widgets na janela

```
Enter your name:
[ QLineEdit ]

Enter your age:
[ QSpinBox ]

[ Send ]
```

---

### ObjectName dos widgets

| Widget      | ObjectName |
| ----------- | ---------- |
| QLabel      | nameLabel  |
| QLineEdit   | nameInput  |
| QSpinBox    | ageInput   |
| QPushButton | sendButton |

---

### Texto mostrado

| Widget  | Text            |
| ------- | --------------- |
| Label 1 | Enter your name |
| Label 2 | Enter your age  |
| Button  | Send            |

---

# 5️⃣ Resultado do programa

Janela:

```
Enter your name:
[ Joaquim ]

Enter your age:
[ 24 ]

[ Send ]
```

Mensagem exibida:

```
Hello Joaquim
Your age is: 24
```

---

# 6️⃣ Estrutura final do projeto

```
Lab2_GUI
│
├── main.cpp
├── mainwindow.cpp
├── mainwindow.h
├── mainwindow.ui
└── CMakeLists.txt
```

---

# 7️⃣ Como explicar para o professor (em russo)

Se ele perguntar o que você fez:

```
В данной работе был создан простой графический интерфейс 
в среде Qt Creator на языке C++.

Были использованы виджеты:
QLabel, QLineEdit, QSpinBox и QPushButton.

При нажатии на кнопку программа получает имя и возраст
пользователя и выводит сообщение с помощью QMessageBox.
```

---

✅ Se quiser, posso também te mostrar **como deixar o layout profissional (Grid Layout)** — isso é algo que **quase sempre dá ponto extra em laboratórios de Qt**.
