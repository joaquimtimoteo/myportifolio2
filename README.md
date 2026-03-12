Pelo seu novo print, o erro está **muito claro**. 👍
Você escreveu **`UI::MainWindow`**, mas o correto no Qt é **`Ui::MainWindow`**.

A diferença é **uma letra minúscula**.

---

# ❌ O que está no seu código

```cpp
ui(new UI::MainWindow)
```

e

```cpp
delete UI;
```

---

# ✅ O correto

```cpp
ui(new Ui::MainWindow)
```

e

```cpp
delete ui;
```

---

# ✔ Seu `mainwindow.cpp` deve ficar assim

Substitua o arquivo inteiro por este código:

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

# ⚠️ Erros que você tinha

| Errado           | Correto          |
| ---------------- | ---------------- |
| `UI::MainWindow` | `Ui::MainWindow` |
| `delete UI`      | `delete ui`      |
| `MainWindows`    | `MainWindow`     |

---

# ▶ Agora faça

1️⃣ **Salvar** o arquivo
2️⃣ Rodar:

```text
Ctrl + R
```

ou clicar no botão:

```
▶ Run
```

---

✅ Se quiser, mande também **um print do `mainwindow.ui`**.
Quero confirmar se **os nomes `nameInput`, `ageInput` e `sendButton` estão corretos**, porque isso é o **último erro que normalmente aparece nesse laboratório**.
