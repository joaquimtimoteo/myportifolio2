Pelo seu print dá para ver **dois erros principais no `mainwindow.cpp`**.

Vou mostrar **exatamente onde corrigir**.

---

# ❌ Erro 1 — Nome errado do construtor

Você escreveu:

```cpp
MainWindow::MainWindows(QWidget *parent)
```

O correto é:

```cpp
MainWindow::MainWindow(QWidget *parent)
```

⚠️ O construtor **deve ter exatamente o mesmo nome da classe**.

---

# ❌ Erro 2 — Você escreveu `UI::MainWindow`

No Qt o namespace é:

```cpp
Ui
```

com **i minúsculo**.

Você escreveu:

```cpp
UI::MainWindow
```

O correto é:

```cpp
Ui::MainWindow
```

---

# ✅ Código correto completo do `mainwindow.cpp`

Substitua **todo o seu arquivo** por este:

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

# ✔ Depois de corrigir

Salve o arquivo e execute:

```
Ctrl + R
```

ou clique:

```
▶ Run
```

---

# ⚠️ Verifique também no Qt Designer

Abra:

```
mainwindow.ui
```

Confirme os **objectName**:

| Widget      | objectName   |
| ----------- | ------------ |
| QLineEdit   | `nameInput`  |
| QSpinBox    | `ageInput`   |
| QPushButton | `sendButton` |

---

✅ Se quiser, mande também **o print do `mainwindow.h`**, porque normalmente aparece **mais um erro pequeno ali**, e eu consigo deixar seu projeto **100% compilando em 1 minuto**.
