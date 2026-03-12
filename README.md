Esse erro é **muito comum no Qt/C++** e normalmente é apenas **um erro de nome da classe**.

O erro:

```
Use of undeclared identifier 'MainWindows'
```

significa:

> O compilador **não conhece essa classe**, porque ela **não existe**.

Provavelmente você escreveu:

```cpp
MainWindows
```

Mas o nome correto é:

```cpp
MainWindow
```

⚠️ **Sem o "s" no final.**

---

# ✅ Como corrigir

## 1️⃣ Abra `mainwindow.cpp`

Procure algo assim:

```cpp
MainWindows::MainWindows(QWidget *parent)
```

ou

```cpp
void MainWindows::on_sendButton_clicked()
```

---

## 2️⃣ Corrija para:

```cpp
MainWindow::MainWindow(QWidget *parent)
```

e

```cpp
void MainWindow::on_sendButton_clicked()
```

---

# ✔ Exemplo correto

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

    QMessageBox::information(this, "Message",
                             "Hello " + name);
}
```

---

# ⚠️ Outra coisa para verificar

Abra **`mainwindow.h`** e confirme que está assim:

```cpp
class MainWindow : public QMainWindow
{
    Q_OBJECT
```

e não:

```cpp
MainWindows
```

---

# 💡 Por que isso acontece?

Quando você cria o projeto, o Qt gera a classe:

```
MainWindow
```

Mas se você escreve manualmente:

```
MainWindows
```

o compilador não encontra essa classe.

---

# ✅ Se quiser, mande aqui:

* o código do **`mainwindow.cpp`**
* ou um **print do erro**

e eu te mostro **exatamente onde corrigir em 10 segundos**.
