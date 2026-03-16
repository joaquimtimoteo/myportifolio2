## Código Completo — Практическая работа 2-1 (9)

São **4 ficheiros**. Cola cada um no Qt Creator.

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

## 📁 `mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QPushButton>
#include <QLabel>
#include <QSignalMapper>

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
    // Задание 1
    void onCheckBoxToggled(bool checked);

    // Задание 2
    void onCellClicked(int id);
    void onClearClicked();

private:
    Ui::MainWindow *ui;

    // Задание 2 — игра
    QPushButton   *buttons[3][3];
    QLabel        *statusLabel;
    QSignalMapper *signalMapper;
    int            currentPlayer;
    int            startPlayer;

    bool checkWinner();
    void resetBoard();
    void updateLabel();

    QWidget *buildWidgetsTab();
    QWidget *buildGameTab();
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
#include <QGridLayout>
#include <QSlider>
#include <QScrollBar>
#include <QSpinBox>
#include <QCheckBox>
#include <QLineEdit>
#include <QPushButton>
#include <QLabel>
#include <QSignalMapper>
#include <QMessageBox>
#include <QGroupBox>
#include <QFrame>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
    , currentPlayer(1)
    , startPlayer(1)
{
    ui->setupUi(this);
    setWindowTitle("Lab 2-1: Виджеты и Крестики-нолики");
    setMinimumSize(520, 480);

    auto *tabs = new QTabWidget(this);
    tabs->addTab(buildWidgetsTab(), "⚙ Виджеты");
    tabs->addTab(buildGameTab(),    "✖ Крестики-нолики");
    setCentralWidget(tabs);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// ─────────────────────────────────────────────
// ЗАДАНИЕ 1 — 5 виджетов с сигнально-слотовыми
// ─────────────────────────────────────────────
QWidget *MainWindow::buildWidgetsTab()
{
    auto *w      = new QWidget;
    auto *layout = new QVBoxLayout(w);
    layout->setSpacing(16);
    layout->setContentsMargins(20, 20, 20, 20);

    // --- Заголовок ---
    auto *title = new QLabel("<b>Сигнально-слотовые соединения</b>");
    title->setStyleSheet("font-size: 14px; color: #1565C0;");
    layout->addWidget(title);

    // --- Группа 1: QSlider ↔ QScrollBar ↔ QSpinBox ---
    auto *group1 = new QGroupBox("1. QSlider ↔ QScrollBar ↔ QSpinBox");
    auto *g1lay  = new QVBoxLayout(group1);

    auto *slider    = new QSlider(Qt::Horizontal);
    auto *scrollBar = new QScrollBar(Qt::Horizontal);
    auto *spinBox   = new QSpinBox;

    slider->setRange(0, 100);
    scrollBar->setRange(0, 100);
    spinBox->setRange(0, 100);

    slider->setFixedHeight(30);
    scrollBar->setFixedHeight(20);

    g1lay->addWidget(new QLabel("QSlider:"));
    g1lay->addWidget(slider);
    g1lay->addWidget(new QLabel("QScrollBar (следует за слайдером):"));
    g1lay->addWidget(scrollBar);

    auto *spinRow = new QHBoxLayout;
    spinRow->addWidget(new QLabel("QSpinBox (синхронизирован):"));
    spinRow->addWidget(spinBox);
    spinRow->addStretch();
    g1lay->addLayout(spinRow);

    // Соединения
    connect(slider,    QOverload<int>::of(&QSlider::valueChanged),
            scrollBar, &QScrollBar::setValue);
    connect(slider,    QOverload<int>::of(&QSlider::valueChanged),
            spinBox,   &QSpinBox::setValue);
    connect(spinBox,   QOverload<int>::of(&QSpinBox::valueChanged),
            slider,    &QSlider::setValue);

    layout->addWidget(group1);

    // --- Группа 2: QCheckBox → QLabel ---
    auto *group2 = new QGroupBox("2. QCheckBox → QLabel");
    auto *g2lay  = new QHBoxLayout(group2);

    auto *checkBox   = new QCheckBox("Активировать");
    auto *checkLabel = new QLabel("Статус: Выкл");
    checkLabel->setStyleSheet("color: red; font-weight: bold;");

    g2lay->addWidget(checkBox);
    g2lay->addWidget(checkLabel);
    g2lay->addStretch();

    connect(checkBox, &QCheckBox::toggled,
            this, &MainWindow::onCheckBoxToggled);

    // Сохраняем указатель через лямбду
    connect(checkBox, &QCheckBox::toggled,
            [checkLabel](bool checked) {
                if (checked) {
                    checkLabel->setText("Статус: Вкл ✓");
                    checkLabel->setStyleSheet("color: green; font-weight: bold;");
                } else {
                    checkLabel->setText("Статус: Выкл");
                    checkLabel->setStyleSheet("color: red; font-weight: bold;");
                }
            });

    layout->addWidget(group2);

    // --- Группа 3: QPushButton → QLineEdit.clear() ---
    auto *group3 = new QGroupBox("3. QPushButton → QLineEdit (очистка)");
    auto *g3lay  = new QHBoxLayout(group3);

    auto *lineEdit  = new QLineEdit;
    lineEdit->setPlaceholderText("Введите текст здесь...");
    auto *clearBtn  = new QPushButton("Очистить");
    clearBtn->setStyleSheet("padding: 4px 12px;");

    g3lay->addWidget(lineEdit);
    g3lay->addWidget(clearBtn);

    connect(clearBtn, &QPushButton::clicked,
            lineEdit, &QLineEdit::clear);

    layout->addWidget(group3);
    layout->addStretch();

    return w;
}

void MainWindow::onCheckBoxToggled(bool /*checked*/)
{
    // Логика вынесена в лямбду выше
}

// ─────────────────────────────────────────────
// ЗАДАНИЕ 2 — КРЕСТИКИ-НОЛИКИ
// ─────────────────────────────────────────────
QWidget *MainWindow::buildGameTab()
{
    auto *w      = new QWidget;
    auto *layout = new QVBoxLayout(w);
    layout->setSpacing(12);
    layout->setContentsMargins(20, 20, 20, 20);

    // --- Метка текущего игрока ---
    statusLabel = new QLabel;
    statusLabel->setAlignment(Qt::AlignCenter);
    statusLabel->setStyleSheet(
        "font-size: 16px; font-weight: bold; "
        "padding: 8px; border-radius: 6px; "
        "background: #E3F2FD; color: #1565C0;");
    updateLabel();
    layout->addWidget(statusLabel);

    // --- Игровая сетка 3x3 ---
    auto *gridFrame  = new QFrame;
    auto *gridLayout = new QGridLayout(gridFrame);
    gridLayout->setSpacing(6);

    signalMapper = new QSignalMapper(this);

    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            buttons[i][j] = new QPushButton;
            buttons[i][j]->setFixedSize(90, 90);
            buttons[i][j]->setStyleSheet(
                "font-size: 28px; font-weight: bold; "
                "background: white; border: 2px solid #BBDEFB; "
                "border-radius: 8px;");
            gridLayout->addWidget(buttons[i][j], i, j);

            int id = i * 3 + j;
            signalMapper->setMapping(buttons[i][j], id);
            connect(buttons[i][j], SIGNAL(clicked()),
                    signalMapper,   SLOT(map()));
        }
    }

    connect(signalMapper, SIGNAL(mapped(int)),
            this,         SLOT(onCellClicked(int)));

    layout->addWidget(gridFrame, 0, Qt::AlignCenter);

    // --- Кнопка Clear ---
    auto *clearBtn = new QPushButton("🔄  Clear (сброс)");
    clearBtn->setFixedHeight(40);
    clearBtn->setStyleSheet(
        "font-size: 13px; font-weight: bold; "
        "background: #EF5350; color: white; "
        "border-radius: 8px; padding: 0 20px;");
    layout->addWidget(clearBtn);
    layout->addStretch();

    connect(clearBtn, &QPushButton::clicked,
            this,     &MainWindow::onClearClicked);

    return w;
}

void MainWindow::onCellClicked(int id)
{
    int row = id / 3;
    int col = id % 3;

    // Ячейка уже занята — игнорируем
    if (!buttons[row][col]->text().isEmpty())
        return;

    // Устанавливаем символ
    if (currentPlayer == 1) {
        buttons[row][col]->setText("X");
        buttons[row][col]->setStyleSheet(
            "font-size: 28px; font-weight: bold; "
            "background: #E3F2FD; color: #1565C0; "
            "border: 2px solid #1565C0; border-radius: 8px;");
    } else {
        buttons[row][col]->setText("O");
        buttons[row][col]->setStyleSheet(
            "font-size: 28px; font-weight: bold; "
            "background: #FFEBEE; color: #C62828; "
            "border: 2px solid #C62828; border-radius: 8px;");
    }

    // Проверка победителя
    if (checkWinner()) {
        QMessageBox::information(this, "Победа! 🎉",
            QString("Победил Player %1!").arg(currentPlayer));
        resetBoard();
        return;
    }

    // Проверка ничьей
    bool full = true;
    for (int i = 0; i < 3; i++)
        for (int j = 0; j < 3; j++)
            if (buttons[i][j]->text().isEmpty()) {
                full = false;
                break;
            }

    if (full) {
        QMessageBox::information(this, "Ничья! 🤝", "Ничья! Никто не победил.");
        resetBoard();
        return;
    }

    // Смена игрока
    currentPlayer = (currentPlayer == 1) ? 2 : 1;
    updateLabel();
}

void MainWindow::onClearClicked()
{
    // Смена первого хода
    startPlayer   = (startPlayer == 1) ? 2 : 1;
    currentPlayer = startPlayer;
    resetBoard();
}

bool MainWindow::checkWinner()
{
    // Проверка строк
    for (int i = 0; i < 3; i++) {
        if (!buttons[i][0]->text().isEmpty() &&
            buttons[i][0]->text() == buttons[i][1]->text() &&
            buttons[i][1]->text() == buttons[i][2]->text())
            return true;
    }
    // Проверка столбцов
    for (int j = 0; j < 3; j++) {
        if (!buttons[0][j]->text().isEmpty() &&
            buttons[0][j]->text() == buttons[1][j]->text() &&
            buttons[1][j]->text() == buttons[2][j]->text())
            return true;
    }
    // Проверка диагоналей
    if (!buttons[0][0]->text().isEmpty() &&
        buttons[0][0]->text() == buttons[1][1]->text() &&
        buttons[1][1]->text() == buttons[2][2]->text())
        return true;

    if (!buttons[0][2]->text().isEmpty() &&
        buttons[0][2]->text() == buttons[1][1]->text() &&
        buttons[1][1]->text() == buttons[2][0]->text())
        return true;

    return false;
}

void MainWindow::resetBoard()
{
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            buttons[i][j]->setText("");
            buttons[i][j]->setStyleSheet(
                "font-size: 28px; font-weight: bold; "
                "background: white; border: 2px solid #BBDEFB; "
                "border-radius: 8px;");
        }
    }
    updateLabel();
}

void MainWindow::updateLabel()
{
    if (currentPlayer == 1) {
        statusLabel->setText("🎮  Ход: Player 1  —  X");
        statusLabel->setStyleSheet(
            "font-size: 16px; font-weight: bold; "
            "padding: 8px; border-radius: 6px; "
            "background: #E3F2FD; color: #1565C0;");
    } else {
        statusLabel->setText("🎮  Ход: Player 2  —  O");
        statusLabel->setStyleSheet(
            "font-size: 16px; font-weight: bold; "
            "padding: 8px; border-radius: 6px; "
            "background: #FFEBEE; color: #C62828;");
    }
}
```

---

## 📁 `CMakeLists.txt`

Verifica que o teu `CMakeLists.txt` tem isto (o Qt Creator gera automaticamente, só confirma):

```cmake
cmake_minimum_required(VERSION 3.16)
project(Lab2GUI VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

set(PROJECT_SOURCES
    main.cpp
    mainwindow.cpp
    mainwindow.h
    mainwindow.ui
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(Lab2GUI MANUAL_FINALIZATION ${PROJECT_SOURCES})
else()
    add_executable(Lab2GUI ${PROJECT_SOURCES})
endif()

target_link_libraries(Lab2GUI PRIVATE Qt${QT_VERSION_MAJOR}::Widgets)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(Lab2GUI)
endif()
```

---

## Resultado esperado

```
┌─────────────────────────────────────────┐
│  ⚙ Виджеты  │  ✖ Крестики-нолики       │
├─────────────────────────────────────────┤
│  🎮 Ход: Player 1 — X                  │
│                                         │
│  [ X ] [   ] [ O ]                     │
│  [   ] [ X ] [   ]                     │
│  [ O ] [   ] [   ]                     │
│                                         │
│        [ 🔄 Clear ]                    │
└─────────────────────────────────────────┘
```

Pressiona **`Ctrl + R`** e manda o print! 🚀
