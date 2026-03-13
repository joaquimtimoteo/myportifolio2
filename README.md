Perfeito! O projeto novo está limpo. Vejo um problema na linha 2 do `mainwindow.cpp`:

❌ Tens: `#include "./ui_mainwindow.h"`  
✅ Deve ser: `#include "ui_mainwindow.h"`

---

## Passo 1 — Corrige o `mainwindow.cpp`

Substitui o conteúdo completo por este:

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QMessageBox>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
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

## Passo 2 — Adiciona o slot no `mainwindow.h`

Na secção `private slots:` adiciona:

```cpp
private slots:
    void on_sendButton_clicked();
```

O teu `mainwindow.h` deve ficar assim:

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>

QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
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
#endif
```

---

## Passo 3 — Configura o `mainwindow.ui`

Abre o `mainwindow.ui` clicando duas vezes nele. No **Qt Designer** arrasta para a janela:

| Widget | objectName |
|---|---|
| `QLineEdit` | `nameInput` |
| `QSpinBox` | `ageInput` |
| `QPushButton` | `sendButton` |

Depois pressiona **`Ctrl + R`** para compilar e rodar! 🚀

Manda print do Designer quando abrires o `.ui`!
