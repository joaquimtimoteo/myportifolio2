Perfeito — isso explica o problema.
Se você **só tem**:

```
main.cpp
mainwindow.cpp
mainwindow.ui
```

então **falta um arquivo muito importante**:

```
mainwindow.h
```

Sem ele o projeto normalmente **não compila corretamente**.

Vamos resolver em **2 minutos**.

---

# 1️⃣ Criar o arquivo `mainwindow.h`

No **Qt Creator**:

1️⃣ Clique com botão direito no projeto
2️⃣ Escolha:

```
Add New...
```

3️⃣ Depois:

```
C++ → C++ Header File
```

4️⃣ Nome do arquivo:

```
mainwindow.h
```

Clique:

```
Next → Finish
```

---

# 2️⃣ Coloque este código dentro do `mainwindow.h`

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

# 3️⃣ Agora verifique o `mainwindow.cpp`

Ele deve estar assim:

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

# 4️⃣ Verifique o `main.cpp`

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

# 5️⃣ Compilar novamente

Agora execute:

```
Ctrl + R
```

ou clique:

```
▶ Run
```

---

# ⚠️ Muito importante

No **Qt Designer (`mainwindow.ui`)** você precisa ter:

| Widget      | objectName   |
| ----------- | ------------ |
| QLineEdit   | `nameInput`  |
| QSpinBox    | `ageInput`   |
| QPushButton | `sendButton` |

---

# ✅ Depois disso seu programa vai funcionar:

```
Enter your name:
[ Joaquim ]

Enter your age:
[ 24 ]

[ Send ]
```

Mensagem:

```
Hello Joaquim
Your age is: 24
```

---

💡 Se quiser, posso também te mostrar **como organizar o layout corretamente (Grid Layout)** — isso é **um detalhe que muitos professores russos exigem no relatório do laboratório**.
