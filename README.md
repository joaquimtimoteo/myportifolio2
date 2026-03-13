## Guia Passo a Passo — Практическая работа 2-2

---

## Passo 1 — Criar novo projeto

No Qt Creator:
```
File → New Project → Qt Widgets Application
```
- **Name:** `Lab2GUI2`
- Clica **Next** até ao fim → **Finish**

---

## Passo 2 — Criar a classe personalizada

Vamos criar um widget chamado `IconLineEdit`.

```
File → New File → C++ → C++ Class → Choose
```
Preenche:
- **Class name:** `IconLineEdit`
- **Base class:** `QWidget`
- Clica **Finish**

Isso cria dois ficheiros: `iconlineedit.h` e `iconlineedit.cpp`

---

## Passo 3 — Código do `iconlineedit.h`

Apaga tudo e cola isto:

```cpp
#ifndef ICONLINEEDIT_H
#define ICONLINEEDIT_H

#include <QWidget>
#include <QLineEdit>
#include <QLabel>
#include <QHBoxLayout>

class IconLineEdit : public QWidget
{
    Q_OBJECT

public:
    explicit IconLineEdit(QWidget *parent = nullptr);
    QString text() const;
    void setText(const QString &text);
    void setPlaceholder(const QString &text);

signals:
    void textChanged(const QString &text);

private slots:
    void onTextChanged(const QString &text);

private:
    QLineEdit  *m_lineEdit;
    QLabel     *m_iconLabel;
    QHBoxLayout *m_layout;

    void updateIcon(const QString &text);
};

#endif
```

---

## Passo 4 — Código do `iconlineedit.cpp`

Apaga tudo e cola isto:

```cpp
#include "iconlineedit.h"

IconLineEdit::IconLineEdit(QWidget *parent)
    : QWidget(parent)
{
    m_layout   = new QHBoxLayout(this);
    m_iconLabel = new QLabel(this);
    m_lineEdit  = new QLineEdit(this);

    m_iconLabel->setFixedSize(24, 24);
    m_iconLabel->setAlignment(Qt::AlignCenter);
    m_iconLabel->setText("🔴"); // ícone inicial = vazio

    m_layout->addWidget(m_iconLabel);
    m_layout->addWidget(m_lineEdit);
    m_layout->setContentsMargins(4, 4, 4, 4);

    setLayout(m_layout);

    connect(m_lineEdit, &QLineEdit::textChanged,
            this, &IconLineEdit::onTextChanged);
}

QString IconLineEdit::text() const
{
    return m_lineEdit->text();
}

void IconLineEdit::setText(const QString &text)
{
    m_lineEdit->setText(text);
}

void IconLineEdit::setPlaceholder(const QString &text)
{
    m_lineEdit->setPlaceholderText(text);
}

void IconLineEdit::onTextChanged(const QString &text)
{
    updateIcon(text);
    emit textChanged(text);
}

void IconLineEdit::updateIcon(const QString &text)
{
    if (text.isEmpty()) {
        m_iconLabel->setText("🔴"); // vazio
    } else {
        m_iconLabel->setText("✅"); // preenchido
    }
}
```

---

## Passo 5 — Código do `mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include "iconlineedit.h"

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
    void onSendClicked();

private:
    Ui::MainWindow *ui;
    IconLineEdit   *m_nameField;
    IconLineEdit   *m_emailField;
};

#endif
```

---

## Passo 6 — Código do `mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QVBoxLayout>
#include <QLabel>
#include <QPushButton>
#include <QMessageBox>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    setWindowTitle("Lab 2-2: Custom Widget");

    QWidget     *central = new QWidget(this);
    QVBoxLayout *layout  = new QVBoxLayout(central);

    QLabel *title = new QLabel("User Form", central);
    title->setStyleSheet("font-size: 16px; font-weight: bold;");

    m_nameField  = new IconLineEdit(central);
    m_nameField->setPlaceholder("Enter your name...");

    m_emailField = new IconLineEdit(central);
    m_emailField->setPlaceholder("Enter your email...");

    QPushButton *btn = new QPushButton("Submit", central);

    layout->addWidget(title);
    layout->addWidget(new QLabel("Name:", central));
    layout->addWidget(m_nameField);
    layout->addWidget(new QLabel("Email:", central));
    layout->addWidget(m_emailField);
    layout->addWidget(btn);
    layout->addStretch();

    central->setLayout(layout);
    setCentralWidget(central);

    connect(btn, &QPushButton::clicked,
            this, &MainWindow::onSendClicked);
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::onSendClicked()
{
    QString name  = m_nameField->text();
    QString email = m_emailField->text();

    if (name.isEmpty() || email.isEmpty()) {
        QMessageBox::warning(this, "Warning", "Please fill all fields!");
        return;
    }

    QMessageBox::information(this, "Success",
        "Name: "  + name  + "\n" +
        "Email: " + email);
}
```

---

## Passo 7 — Compilar e rodar

Pressiona **`Ctrl + R`**

---

## O resultado esperado

```
┌─────────────────────────────┐
│   User Form                 │
│                             │
│  Name:                      │
│  🔴 [              ]        │ ← vazio
│                             │
│  Email:                     │
│  ✅ [john@email.com]        │ ← preenchido
│                             │
│  [ Submit ]                 │
└─────────────────────────────┘
```

---

Manda um print quando compilar! Se tiver algum erro eu resolvo. 🚀
