## Código Completo — Lab 2-2 (Todas as Tarefas)

São **7 ficheiros** no total. Cola cada um com cuidado.

---

## 📁 `iconizedlineedit.h`

```cpp
#ifndef ICONIZEDLINEEDIT_H
#define ICONIZEDLINEEDIT_H

#include <QWidget>
#include <QLineEdit>
#include <QLabel>
#include <QHBoxLayout>
#include <QValidator>

class IconizedLineEdit : public QWidget
{
    Q_OBJECT

public:
    enum IconMode {
        ModeStatic,     // ícone fixo
        ModeValidation, // muda conforme validação
        ModeFile        // clique abre diálogo de ficheiro
    };

    explicit IconizedLineEdit(QWidget *parent = nullptr);

    QString text() const;
    void setText(const QString &text);
    void setPlaceholderText(const QString &text);
    void setEchoMode(QLineEdit::EchoMode mode);
    void setValidator(const QValidator *validator);
    void setIconMode(IconMode mode);
    void setStaticIcon(const QString &icon);

signals:
    void textChanged(const QString &text);
    void iconClicked();

private slots:
    void onTextChanged(const QString &text);
    void onIconClicked();

private:
    QLineEdit   *m_lineEdit;
    QLabel      *m_iconLabel;
    QHBoxLayout *m_layout;
    IconMode     m_mode;

    void updateIcon(const QString &text);
};

#endif
```

---

## 📁 `iconizedlineedit.cpp`

```cpp
#include "iconizedlineedit.h"
#include <QMouseEvent>

// Subclasse interna para detectar clique no ícone
class ClickableLabel : public QLabel {
    Q_OBJECT
public:
    explicit ClickableLabel(QWidget *parent = nullptr) : QLabel(parent) {
        setCursor(Qt::PointingHandCursor);
    }
signals:
    void clicked();
protected:
    void mousePressEvent(QMouseEvent *) override {
        emit clicked();
    }
};

#include "iconizedlineedit.moc"

IconizedLineEdit::IconizedLineEdit(QWidget *parent)
    : QWidget(parent)
    , m_mode(ModeValidation)
{
    m_layout = new QHBoxLayout(this);
    m_layout->setContentsMargins(2, 2, 2, 2);
    m_layout->setSpacing(4);

    auto *icon = new ClickableLabel(this);
    m_iconLabel = icon;
    m_iconLabel->setFixedSize(28, 28);
    m_iconLabel->setAlignment(Qt::AlignCenter);
    m_iconLabel->setText("🔴");
    m_iconLabel->setStyleSheet("font-size: 16px;");

    m_lineEdit = new QLineEdit(this);

    m_layout->addWidget(m_iconLabel);
    m_layout->addWidget(m_lineEdit);
    setLayout(m_layout);

    connect(icon, &ClickableLabel::clicked,
            this, &IconizedLineEdit::onIconClicked);
    connect(m_lineEdit, &QLineEdit::textChanged,
            this, &IconizedLineEdit::onTextChanged);
}

QString IconizedLineEdit::text() const {
    return m_lineEdit->text();
}

void IconizedLineEdit::setText(const QString &text) {
    m_lineEdit->setText(text);
}

void IconizedLineEdit::setPlaceholderText(const QString &text) {
    m_lineEdit->setPlaceholderText(text);
}

void IconizedLineEdit::setEchoMode(QLineEdit::EchoMode mode) {
    m_lineEdit->setEchoMode(mode);
}

void IconizedLineEdit::setValidator(const QValidator *v) {
    m_lineEdit->setValidator(v);
}

void IconizedLineEdit::setIconMode(IconMode mode) {
    m_mode = mode;
    if (mode == ModeFile) {
        m_iconLabel->setText("📂");
        m_iconLabel->setToolTip("Clique para escolher ficheiro");
    }
}

void IconizedLineEdit::setStaticIcon(const QString &icon) {
    m_mode = ModeStatic;
    m_iconLabel->setText(icon);
}

void IconizedLineEdit::onTextChanged(const QString &text) {
    if (m_mode == ModeValidation) {
        updateIcon(text);
    }
    emit textChanged(text);
}

void IconizedLineEdit::onIconClicked() {
    emit iconClicked();
}

void IconizedLineEdit::updateIcon(const QString &text) {
    const QValidator *v = m_lineEdit->validator();
    if (text.isEmpty()) {
        m_iconLabel->setText("🔴");
        return;
    }
    if (v) {
        int pos = 0;
        QString t = text;
        if (v->validate(t, pos) == QValidator::Acceptable) {
            m_iconLabel->setText("✅");
        } else {
            m_iconLabel->setText("⚠️");
        }
    } else {
        m_iconLabel->setText("✅");
    }
}
```

---

## 📁 `ledindicator.h`

```cpp
#ifndef LEDINDICATOR_H
#define LEDINDICATOR_H

#include <QWidget>

class LedIndicator : public QWidget
{
    Q_OBJECT

public:
    enum State {
        Off,          // apagado
        On,           // ligado
        Intermediate  // intermédio (tristate)
    };

    explicit LedIndicator(QWidget *parent = nullptr);

    void setTristate(bool enable);
    void setState(State state);
    State state() const;

    QSize sizeHint() const override;

protected:
    void paintEvent(QPaintEvent *event) override;
    void mousePressEvent(QMouseEvent *event) override;

signals:
    void stateChanged(State state);

private:
    State m_state;
    bool  m_tristate;
};

#endif
```

---

## 📁 `ledindicator.cpp`

```cpp
#include "ledindicator.h"
#include <QPainter>
#include <QRadialGradient>

LedIndicator::LedIndicator(QWidget *parent)
    : QWidget(parent)
    , m_state(Off)
    , m_tristate(false)
{
    setFixedSize(32, 32);
    setCursor(Qt::PointingHandCursor);
}

void LedIndicator::setTristate(bool enable) {
    m_tristate = enable;
}

void LedIndicator::setState(State state) {
    if (!m_tristate && state == Intermediate)
        return;
    m_state = state;
    update();
    emit stateChanged(m_state);
}

LedIndicator::State LedIndicator::state() const {
    return m_state;
}

QSize LedIndicator::sizeHint() const {
    return QSize(32, 32);
}

void LedIndicator::paintEvent(QPaintEvent *) {
    QPainter p(this);
    p.setRenderHint(QPainter::Antialiasing);

    QColor color;
    switch (m_state) {
        case Off:          color = QColor(80, 80, 80);    break;
        case On:           color = QColor(50, 200, 50);   break;
        case Intermediate: color = QColor(255, 165, 0);   break;
    }

    // Sombra exterior
    p.setBrush(color.darker(150));
    p.setPen(Qt::NoPen);
    p.drawEllipse(2, 2, 28, 28);

    // LED principal
    QRadialGradient grad(12, 10, 14);
    grad.setColorAt(0, color.lighter(160));
    grad.setColorAt(1, color.darker(120));
    p.setBrush(grad);
    p.setPen(QPen(color.darker(180), 1));
    p.drawEllipse(3, 3, 26, 26);

    // Brilho
    p.setBrush(QColor(255, 255, 255, 80));
    p.setPen(Qt::NoPen);
    p.drawEllipse(8, 6, 10, 8);
}

void LedIndicator::mousePressEvent(QMouseEvent *) {
    // Cicla pelos estados
    if (m_tristate) {
        switch (m_state) {
            case Off:          setState(On);           break;
            case On:           setState(Intermediate); break;
            case Intermediate: setState(Off);          break;
        }
    } else {
        setState(m_state == Off ? On : Off);
    }
}
```

---

## 📁 `mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QLabel>
#include "iconizedlineedit.h"
#include "ledindicator.h"

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
    // Tarefa 1+2: Login
    void onLoginClicked();

    // Tarefa 3: Imagem
    void onChooseFile();
    void onLoadImage();

    // Tarefa 4: LED
    void onLedStateChanged(LedIndicator::State state);

private:
    Ui::MainWindow *ui;

    // Tarefa 1+2
    IconizedLineEdit *m_userField;
    IconizedLineEdit *m_passField;
    QLabel           *m_loginStatus;

    // Tarefa 3
    IconizedLineEdit *m_pathField;
    QLabel           *m_imageLabel;

    // Tarefa 4
    LedIndicator *m_led;
    QLabel       *m_ledStatus;

    QWidget *buildLoginTab();
    QWidget *buildImageTab();
    QWidget *buildLedTab();
};

#endif
```

---

## 📁 `mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QTabWidget>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QLabel>
#include <QPushButton>
#include <QMessageBox>
#include <QFileDialog>
#include <QPixmap>
#include <QRegularExpressionValidator>
#include <QRegularExpression>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    setWindowTitle("Lab 2-2: Custom Qt Classes");
    setMinimumSize(500, 400);

    auto *tabs = new QTabWidget(this);
    tabs->addTab(buildLoginTab(), "🔐 Tarefa 1-2: Login");
    tabs->addTab(buildImageTab(), "🖼  Tarefa 3: Imagem");
    tabs->addTab(buildLedTab(),   "💡 Tarefa 4: LED");

    setCentralWidget(tabs);
}

MainWindow::~MainWindow() {
    delete ui;
}

// ─── TAREFA 1 + 2 ────────────────────────────────────────────────
QWidget *MainWindow::buildLoginTab() {
    auto *w      = new QWidget;
    auto *layout = new QVBoxLayout(w);
    layout->setSpacing(12);
    layout->setContentsMargins(24, 24, 24, 24);

    auto *title = new QLabel("<b>User Login</b>");
    title->setStyleSheet("font-size: 15px;");

    // Username: mínimo 3 caracteres, só letras/números
    m_userField = new IconizedLineEdit(w);
    m_userField->setPlaceholderText("Username (min. 3 chars)...");
    m_userField->setIconMode(IconizedLineEdit::ModeValidation);
    auto *userVal = new QRegularExpressionValidator(
        QRegularExpression("[A-Za-z0-9]{3,}"), w);
    m_userField->setValidator(userVal);

    // Password: mínimo 6 caracteres
    m_passField = new IconizedLineEdit(w);
    m_passField->setPlaceholderText("Password (min. 6 chars)...");
    m_passField->setEchoMode(QLineEdit::Password);
    m_passField->setIconMode(IconizedLineEdit::ModeValidation);
    auto *passVal = new QRegularExpressionValidator(
        QRegularExpression(".{6,}"), w);
    m_passField->setValidator(passVal);

    auto *btn = new QPushButton("Login", w);
    btn->setStyleSheet("padding: 6px 20px; font-size: 13px;");

    m_loginStatus = new QLabel("", w);
    m_loginStatus->setAlignment(Qt::AlignCenter);

    layout->addWidget(title);
    layout->addWidget(new QLabel("Username:"));
    layout->addWidget(m_userField);
    layout->addWidget(new QLabel("Password:"));
    layout->addWidget(m_passField);
    layout->addWidget(btn);
    layout->addWidget(m_loginStatus);
    layout->addStretch();

    connect(btn, &QPushButton::clicked,
            this, &MainWindow::onLoginClicked);
    return w;
}

void MainWindow::onLoginClicked() {
    QString user = m_userField->text();
    QString pass = m_passField->text();

    if (user.length() < 3) {
        m_loginStatus->setStyleSheet("color: red;");
        m_loginStatus->setText("❌ Username deve ter mínimo 3 caracteres!");
        return;
    }
    if (pass.length() < 6) {
        m_loginStatus->setStyleSheet("color: red;");
        m_loginStatus->setText("❌ Password deve ter mínimo 6 caracteres!");
        return;
    }
    m_loginStatus->setStyleSheet("color: green;");
    m_loginStatus->setText("✅ Login bem-sucedido! Bem-vindo, " + user + "!");
}

// ─── TAREFA 3 ────────────────────────────────────────────────────
QWidget *MainWindow::buildImageTab() {
    auto *w      = new QWidget;
    auto *layout = new QVBoxLayout(w);
    layout->setSpacing(12);
    layout->setContentsMargins(24, 24, 24, 24);

    auto *title = new QLabel("<b>Image Viewer</b>");
    title->setStyleSheet("font-size: 15px;");

    m_pathField = new IconizedLineEdit(w);
    m_pathField->setPlaceholderText("Caminho da imagem...");
    m_pathField->setIconMode(IconizedLineEdit::ModeFile);

    auto *btnLoad = new QPushButton("Carregar Imagem", w);

    m_imageLabel = new QLabel(w);
    m_imageLabel->setAlignment(Qt::AlignCenter);
    m_imageLabel->setMinimumHeight(200);
    m_imageLabel->setStyleSheet(
        "border: 1px dashed gray; background: #f9f9f9;");
    m_imageLabel->setText("[ Nenhuma imagem carregada ]");

    layout->addWidget(title);
    layout->addWidget(new QLabel("Caminho do ficheiro:"));
    layout->addWidget(m_pathField);
    layout->addWidget(btnLoad);
    layout->addWidget(m_imageLabel);

    connect(m_pathField, &IconizedLineEdit::iconClicked,
            this, &MainWindow::onChooseFile);
    connect(btnLoad, &QPushButton::clicked,
            this, &MainWindow::onLoadImage);
    return w;
}

void MainWindow::onChooseFile() {
    QString path = QFileDialog::getOpenFileName(
        this, "Escolher Imagem", "",
        "Images (*.png *.jpg *.jpeg *.bmp *.gif)");
    if (!path.isEmpty()) {
        m_pathField->setText(path);
        onLoadImage();
    }
}

void MainWindow::onLoadImage() {
    QString path = m_pathField->text();
    if (path.isEmpty()) {
        QMessageBox::warning(this, "Aviso", "Introduza o caminho da imagem!");
        return;
    }
    QPixmap pix(path);
    if (pix.isNull()) {
        m_imageLabel->setText("❌ Não foi possível carregar a imagem!");
        return;
    }
    m_imageLabel->setPixmap(
        pix.scaled(m_imageLabel->size(),
                   Qt::KeepAspectRatio,
                   Qt::SmoothTransformation));
}

// ─── TAREFA 4 ────────────────────────────────────────────────────
QWidget *MainWindow::buildLedTab() {
    auto *w      = new QWidget;
    auto *layout = new QVBoxLayout(w);
    layout->setSpacing(16);
    layout->setContentsMargins(24, 24, 24, 24);

    auto *title = new QLabel("<b>LED Indicator — 3 States</b>");
    title->setStyleSheet("font-size: 15px;");

    m_led = new LedIndicator(w);
    m_led->setTristate(true); // ativa o 3º estado

    m_ledStatus = new QLabel("Estado: OFF", w);
    m_ledStatus->setStyleSheet("font-size: 13px;");
    m_ledStatus->setAlignment(Qt::AlignCenter);

    auto *info = new QLabel(
        "🖱 Clique no LED para mudar o estado:\n"
        "OFF → ON → INTERMÉDIO → OFF ...", w);
    info->setStyleSheet("color: gray; font-size: 12px;");
    info->setAlignment(Qt::AlignCenter);

    // Botões manuais
    auto *btnOff  = new QPushButton("Set OFF", w);
    auto *btnOn   = new QPushButton("Set ON", w);
    auto *btnMid  = new QPushButton("Set INTERMÉDIO", w);

    auto *btnRow = new QHBoxLayout;
    btnRow->addWidget(btnOff);
    btnRow->addWidget(btnOn);
    btnRow->addWidget(btnMid);

    layout->addWidget(title);
    layout->addWidget(m_led, 0, Qt::AlignCenter);
    layout->addWidget(m_ledStatus);
    layout->addWidget(info);
    layout->addLayout(btnRow);
    layout->addStretch();

    connect(m_led, &LedIndicator::stateChanged,
            this, &MainWindow::onLedStateChanged);
    connect(btnOff,  &QPushButton::clicked, [this](){
        m_led->setState(LedIndicator::Off); });
    connect(btnOn,   &QPushButton::clicked, [this](){
        m_led->setState(LedIndicator::On); });
    connect(btnMid,  &QPushButton::clicked, [this](){
        m_led->setState(LedIndicator::Intermediate); });

    return w;
}

void MainWindow::onLedStateChanged(LedIndicator::State state) {
    switch (state) {
        case LedIndicator::Off:
            m_ledStatus->setStyleSheet("color: gray; font-size: 13px;");
            m_ledStatus->setText("Estado: ⚫ OFF");
            break;
        case LedIndicator::On:
            m_ledStatus->setStyleSheet("color: green; font-size: 13px;");
            m_ledStatus->setText("Estado: 🟢 ON");
            break;
        case LedIndicator::Intermediate:
            m_ledStatus->setStyleSheet("color: orange; font-size: 13px;");
            m_ledStatus->setText("Estado: 🟡 INTERMÉDIO");
            break;
    }
}
```

---

## 📁 `main.cpp`

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

## ⚠️ Nota importante — ficheiro `.moc`

O `iconizedlineedit.cpp` usa `#include "iconizedlineedit.moc"` porque a classe `ClickableLabel` está definida dentro do `.cpp`. Isto é necessário para o `Q_OBJECT` funcionar. O Qt Creator gera este ficheiro automaticamente ao compilar.

---

## Resultado esperado

```
┌──────────────────────────────────────┐
│ 🔐 Tarefa 1-2  │ 🖼 Tarefa 3 │ 💡 Tarefa 4 │
├──────────────────────────────────────┤
│  Username:  🔴 [          ]          │
│  Password:  ✅ [••••••••• ]          │
│             [ Login ]                │
│  ✅ Login bem-sucedido!              │
└──────────────────────────────────────┘
```

Pressiona **`Ctrl + R`** e manda o print! 🚀
