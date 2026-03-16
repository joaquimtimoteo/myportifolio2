## Código Completo — Практическая работа 2-3

São **7 ficheiros**. Cola cada um com cuidado.

> ⚠️ Para o QCustomPlot: descarrega `qcustomplot.h` e `qcustomplot.cpp` em [qcustomplot.com](https://www.qcustomplot.com) e coloca na pasta do projeto.

---

## 📁 `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.16)
project(Lab2GUI3 VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets PrintSupport)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets PrintSupport)

set(PROJECT_SOURCES
    main.cpp
    mainwindow.cpp
    mainwindow.h
    mainwindow.ui
    settingsdialog.cpp
    settingsdialog.h
    settingsdialog.ui
    qcustomplot.cpp
    qcustomplot.h
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(Lab2GUI3 MANUAL_FINALIZATION ${PROJECT_SOURCES})
else()
    add_executable(Lab2GUI3 ${PROJECT_SOURCES})
endif()

target_link_libraries(Lab2GUI3 PRIVATE
    Qt${QT_VERSION_MAJOR}::Widgets
    Qt${QT_VERSION_MAJOR}::PrintSupport
)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(Lab2GUI3)
endif()
```

---

## 📁 `main.cpp`

```cpp
#include "mainwindow.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    a.setApplicationName("Lab2GUI3");
    a.setOrganizationName("QtLab");

    MainWindow w;
    w.show();
    return a.exec();
}
```

---

## 📁 `settingsdialog.h`

```cpp
#ifndef SETTINGSDIALOG_H
#define SETTINGSDIALOG_H

#include <QDialog>
#include <QSettings>

namespace Ui { class SettingsDialog; }

class SettingsDialog : public QDialog
{
    Q_OBJECT

public:
    explicit SettingsDialog(QWidget *parent = nullptr);
    ~SettingsDialog();

    bool saveGeometry() const;

private slots:
    void onAccepted();

private:
    Ui::SettingsDialog *ui;
    void loadSettings();
    void saveSettings();
};

#endif
```

---

## 📁 `settingsdialog.cpp`

```cpp
#include "settingsdialog.h"
#include "ui_settingsdialog.h"
#include <QSettings>

SettingsDialog::SettingsDialog(QWidget *parent)
    : QDialog(parent)
    , ui(new Ui::SettingsDialog)
{
    ui->setupUi(this);
    setWindowTitle("Настройки");
    setModal(true);
    loadSettings();

    connect(ui->buttonBox, &QDialogButtonBox::accepted,
            this, &SettingsDialog::onAccepted);
    connect(ui->buttonBox, &QDialogButtonBox::rejected,
            this, &QDialog::reject);
}

SettingsDialog::~SettingsDialog()
{
    delete ui;
}

bool SettingsDialog::saveGeometry() const
{
    return ui->checkSaveGeometry->isChecked();
}

void SettingsDialog::loadSettings()
{
    QSettings s;
    ui->checkSaveGeometry->setChecked(
        s.value("saveGeometry", false).toBool());
}

void SettingsDialog::saveSettings()
{
    QSettings s;
    s.setValue("saveGeometry", ui->checkSaveGeometry->isChecked());
}

void SettingsDialog::onAccepted()
{
    saveSettings();
    accept();
}
```

---

## 📁 `settingsdialog.ui`

Cria o ficheiro `settingsdialog.ui` com este conteúdo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>SettingsDialog</class>
 <widget class="QDialog" name="SettingsDialog">
  <property name="geometry">
   <rect><x>0</x><y>0</y><width>360</width><height>180</height></rect>
  </property>
  <property name="windowTitle"><string>Настройки</string></property>
  <layout class="QVBoxLayout">
   <item>
    <widget class="QGroupBox">
     <property name="title"><string>Параметры окна</string></property>
     <layout class="QVBoxLayout">
      <item>
       <widget class="QCheckBox" name="checkSaveGeometry">
        <property name="text">
         <string>Сохранять геометрию главного окна</string>
        </property>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
   <item>
    <widget class="QDialogButtonBox" name="buttonBox">
     <property name="standardButtons">
      <set>QDialogButtonBox::Cancel|QDialogButtonBox::Ok</set>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
</ui>
```

---

## 📁 `mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QSettings>
#include <QLabel>
#include "qcustomplot.h"

QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

protected:
    void closeEvent(QCloseEvent *event) override;

private slots:
    // Tarefa 1 — StatusBar cursor
    void onCursorPositionChanged();

    // Tarefa 2 — Settings
    void openSettings();

    // Tarefa 4/5 — Graph
    void loadFileAndPlot();

    // Tarefa 6 — QCustomPlot demo
    void plotDemo();

private:
    Ui::MainWindow   *ui;
    QLabel           *cursorLabel;
    QCustomPlot      *plot;

    void setupMenuAndToolbar();
    void setupStatusBar();
    void setupPlot();
    void loadGeometry();
    void saveGeometryIfNeeded();
    void plotDataFromFile(const QString &filePath);
};

#endif
```

---

## 📁 `mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include "settingsdialog.h"

#include <QMenuBar>
#include <QToolBar>
#include <QStatusBar>
#include <QAction>
#include <QFileDialog>
#include <QTextStream>
#include <QMessageBox>
#include <QSplitter>
#include <QVBoxLayout>
#include <QSettings>
#include <QCloseEvent>
#include <cmath>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    setWindowTitle("Lab 2-3: Приложение с GUI");
    setMinimumSize(800, 550);

    setupMenuAndToolbar();
    setupStatusBar();
    setupPlot();
    loadGeometry();

    // Tarefa 1 — сигнал изменения позиции курсора
    connect(ui->textEditor,
            &QPlainTextEdit::cursorPositionChanged,
            this,
            &MainWindow::onCursorPositionChanged);

    // Начальное состояние курсора
    onCursorPositionChanged();

    // Демо-график при запуске
    plotDemo();
}

MainWindow::~MainWindow()
{
    delete ui;
}

// ─────────────────────────────────────────
// МЕНЮ И ПАНЕЛЬ ИНСТРУМЕНТОВ
// ─────────────────────────────────────────
void MainWindow::setupMenuAndToolbar()
{
    // ── Меню Файл ──
    QMenu *fileMenu = menuBar()->addMenu("Файл");

    QAction *actLoad = new QAction("📂  Загрузить данные...", this);
    actLoad->setShortcut(QKeySequence::Open);
    actLoad->setStatusTip("Загрузить данные из файла и построить график");
    fileMenu->addAction(actLoad);

    fileMenu->addSeparator();

    QAction *actExit = new QAction("Выход", this);
    actExit->setShortcut(QKeySequence::Quit);
    fileMenu->addAction(actExit);

    // ── Меню Настройки ──
    QMenu *settingsMenu = menuBar()->addMenu("Настройки");
    QAction *actSettings = new QAction("⚙  Параметры...", this);
    settingsMenu->addAction(actSettings);

    // ── Панель инструментов ──
    QToolBar *toolBar = addToolBar("Главная");
    toolBar->setMovable(false);
    toolBar->addAction(actLoad);
    toolBar->addSeparator();
    toolBar->addAction(actSettings);

    // ── Соединения ──
    connect(actLoad,     &QAction::triggered, this, &MainWindow::loadFileAndPlot);
    connect(actExit,     &QAction::triggered, this, &QMainWindow::close);
    connect(actSettings, &QAction::triggered, this, &MainWindow::openSettings);
}

// ─────────────────────────────────────────
// СТАТУС БАР
// ─────────────────────────────────────────
void MainWindow::setupStatusBar()
{
    cursorLabel = new QLabel("Стр: 1  Кол: 1");
    cursorLabel->setStyleSheet("padding: 0 8px;");
    statusBar()->addPermanentWidget(cursorLabel);
    statusBar()->showMessage("Готово", 3000);
}

// ─────────────────────────────────────────
// TAREFA 1 — Позиция курсора
// ─────────────────────────────────────────
void MainWindow::onCursorPositionChanged()
{
    QTextCursor cursor = ui->textEditor->textCursor();
    int line = cursor.blockNumber() + 1;
    int col  = cursor.columnNumber() + 1;

    cursorLabel->setText(
        QString("Стр: %1   Кол: %2").arg(line).arg(col));

    statusBar()->showMessage(
        QString("Позиция курсора: строка %1, столбец %2")
            .arg(line).arg(col), 2000);
}

// ─────────────────────────────────────────
// TAREFA 2 — Геометрия окна
// ─────────────────────────────────────────
void MainWindow::loadGeometry()
{
    QSettings s;
    if (s.value("saveGeometry", false).toBool()) {
        restoreGeometry(s.value("windowGeometry").toByteArray());
        restoreState(s.value("windowState").toByteArray());
    }
}

void MainWindow::saveGeometryIfNeeded()
{
    QSettings s;
    if (s.value("saveGeometry", false).toBool()) {
        s.setValue("windowGeometry", saveGeometry());
        s.setValue("windowState",    saveState());
    }
}

void MainWindow::closeEvent(QCloseEvent *event)
{
    saveGeometryIfNeeded();
    event->accept();
}

// ─────────────────────────────────────────
// НАСТРОЙКИ
// ─────────────────────────────────────────
void MainWindow::openSettings()
{
    SettingsDialog dlg(this);
    dlg.exec();
    statusBar()->showMessage("Настройки сохранены", 2000);
}

// ─────────────────────────────────────────
// TAREFA 4/5 — График из файла
// ─────────────────────────────────────────
void MainWindow::setupPlot()
{
    plot = new QCustomPlot(ui->plotWidget);
    auto *layout = new QVBoxLayout(ui->plotWidget);
    layout->setContentsMargins(0, 0, 0, 0);
    layout->addWidget(plot);

    plot->setBackground(QColor("#FAFAFA"));
    plot->xAxis->setLabel("X");
    plot->yAxis->setLabel("Y");
    plot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}

void MainWindow::loadFileAndPlot()
{
    QString path = QFileDialog::getOpenFileName(
        this, "Загрузить данные", "",
        "Text files (*.txt *.csv);;All files (*)");

    if (path.isEmpty()) return;

    plotDataFromFile(path);
    statusBar()->showMessage("Файл загружен: " + path, 3000);
}

void MainWindow::plotDataFromFile(const QString &filePath)
{
    QFile file(filePath);
    if (!file.open(QIODevice::ReadOnly | QIODevice::Text)) {
        QMessageBox::warning(this, "Ошибка",
            "Не удалось открыть файл:\n" + filePath);
        return;
    }

    QVector<double> xData, yData;
    QTextStream in(&file);

    while (!in.atEnd()) {
        QString line = in.readLine().trimmed();
        if (line.isEmpty() || line.startsWith('#')) continue;

        // Поддержка форматов: "x y" или "x,y" или "x;y"
        line.replace(',', ' ');
        line.replace(';', ' ');
        QStringList parts = line.split(' ', Qt::SkipEmptyParts);

        if (parts.size() >= 2) {
            bool okX, okY;
            double x = parts[0].toDouble(&okX);
            double y = parts[1].toDouble(&okY);
            if (okX && okY) {
                xData.append(x);
                yData.append(y);
            }
        }
    }
    file.close();

    if (xData.isEmpty()) {
        QMessageBox::warning(this, "Ошибка",
            "Файл не содержит корректных данных.\n"
            "Формат: x y (по одной паре на строку)");
        return;
    }

    // Строим график
    plot->clearGraphs();
    plot->addGraph();
    plot->graph(0)->setData(xData, yData);
    plot->graph(0)->setPen(QPen(QColor("#1565C0"), 2));
    plot->graph(0)->setName("Данные из файла");
    plot->rescaleAxes();
    plot->replot();

    statusBar()->showMessage(
        QString("График построен: %1 точек").arg(xData.size()), 4000);
}

// ─────────────────────────────────────────
// TAREFA 6 — QCustomPlot демо (sin/cos)
// ─────────────────────────────────────────
void MainWindow::plotDemo()
{
    QVector<double> x(201), sinY(201), cosY(201);
    for (int i = 0; i <= 200; i++) {
        x[i]    = -M_PI + i * (2 * M_PI / 200.0);
        sinY[i] = std::sin(x[i]);
        cosY[i] = std::cos(x[i]);
    }

    plot->clearGraphs();

    // График sin(x)
    plot->addGraph();
    plot->graph(0)->setData(x, sinY);
    plot->graph(0)->setPen(QPen(QColor("#1565C0"), 2));
    plot->graph(0)->setName("sin(x)");

    // График cos(x)
    plot->addGraph();
    plot->graph(1)->setData(x, cosY);
    plot->graph(1)->setPen(QPen(QColor("#C62828"), 2));
    plot->graph(1)->setName("cos(x)");

    plot->legend->setVisible(true);
    plot->xAxis->setLabel("X");
    plot->yAxis->setLabel("Y");
    plot->xAxis->setRange(-M_PI, M_PI);
    plot->yAxis->setRange(-1.2, 1.2);
    plot->replot();
}
```

---

## 📁 `mainwindow.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>MainWindow</class>
 <widget class="QMainWindow" name="MainWindow">
  <property name="geometry">
   <rect><x>0</x><y>0</y><width>900</width><height>600</height></rect>
  </property>
  <property name="windowTitle"><string>Lab 2-3</string></property>
  <widget class="QWidget" name="centralwidget">
   <layout class="QHBoxLayout" name="horizontalLayout">
    <item>
     <widget class="QSplitter" name="splitter">
      <property name="orientation"><enum>Qt::Horizontal</enum></property>
      <!-- Lado esquerdo: abas -->
      <widget class="QTabWidget" name="tabWidget">
       <!-- Aba 1: Editor de texto (Tarefa 1) -->
       <widget class="QWidget" name="tabEditor">
        <attribute name="title"><string>Редактор</string></attribute>
        <layout class="QVBoxLayout">
         <item>
          <widget class="QPlainTextEdit" name="textEditor">
           <property name="placeholderText">
            <string>Введите текст здесь...
Позиция курсора отображается в статус-баре.</string>
           </property>
          </widget>
         </item>
        </layout>
       </widget>
       <!-- Aba 2: 5 widgets sincronizados (Tarefa 3) -->
       <widget class="QWidget" name="tabWidgets">
        <attribute name="title"><string>Виджеты</string></attribute>
        <layout class="QVBoxLayout">
         <item>
          <widget class="QGroupBox" name="groupBox">
           <property name="title"><string>Slider ↔ ScrollBar ↔ SpinBox</string></property>
           <layout class="QVBoxLayout">
            <item><widget class="QSlider" name="slider">
              <property name="orientation"><enum>Qt::Horizontal</enum></property>
              <property name="maximum"><number>100</number></property>
            </widget></item>
            <item><widget class="QScrollBar" name="scrollBar">
              <property name="orientation"><enum>Qt::Horizontal</enum></property>
              <property name="maximum"><number>100</number></property>
            </widget></item>
            <item>
             <layout class="QHBoxLayout">
              <item><widget class="QLabel"><property name="text"><string>Значение:</string></property></widget></item>
              <item><widget class="QSpinBox" name="spinBox">
                <property name="maximum"><number>100</number></property>
              </widget></item>
              <item><spacer><property name="orientation"><enum>Qt::Horizontal</enum></property></spacer></item>
             </layout>
            </item>
           </layout>
          </widget>
         </item>
         <item>
          <widget class="QGroupBox">
           <property name="title"><string>CheckBox → Label</string></property>
           <layout class="QHBoxLayout">
            <item><widget class="QCheckBox" name="checkBox">
              <property name="text"><string>Активировать</string></property>
            </widget></item>
            <item><widget class="QLabel" name="statusLbl">
              <property name="text"><string>Статус: Выкл</string></property>
              <property name="styleSheet"><string>color: red; font-weight: bold;</string></property>
            </widget></item>
            <item><spacer><property name="orientation"><enum>Qt::Horizontal</enum></property></spacer></item>
           </layout>
          </widget>
         </item>
         <item><spacer><property name="orientation"><enum>Qt::Vertical</enum></property></spacer></item>
        </layout>
       </widget>
      </widget>
      <!-- Lado direito: gráfico -->
      <widget class="QWidget" name="plotWidget"/>
     </widget>
    </item>
   </layout>
  </widget>
 </widget>
</ui>
```

---

## ⚠️ Conectar os widgets da Tarefa 3 no `mainwindow.cpp`

No construtor do `MainWindow`, **após** `ui->setupUi(this)`, adiciona:

```cpp
// ── Tarefa 3: sinais/slots pelo Designer (complemento via código) ──
connect(ui->slider,   QOverload<int>::of(&QSlider::valueChanged),
        ui->scrollBar, &QScrollBar::setValue);
connect(ui->slider,   QOverload<int>::of(&QSlider::valueChanged),
        ui->spinBox,   &QSpinBox::setValue);
connect(ui->spinBox,  QOverload<int>::of(&QSpinBox::valueChanged),
        ui->slider,    &QSlider::setValue);
connect(ui->checkBox, &QCheckBox::toggled,
        [this](bool checked) {
            if (checked) {
                ui->statusLbl->setText("Статус: Вкл ✓");
                ui->statusLbl->setStyleSheet("color:green;font-weight:bold;");
            } else {
                ui->statusLbl->setText("Статус: Выкл");
                ui->statusLbl->setStyleSheet("color:red;font-weight:bold;");
            }
        });
```

---

## 📄 Formato do ficheiro de dados (Tarefa 5)

Cria um ficheiro `data.txt` para testar:

```
# Dados de exemplo (x y)
0.0  0.0
1.0  1.0
2.0  4.0
3.0  9.0
4.0  16.0
5.0  25.0
```

---

## Resultado esperado

```
┌──────────────────────────────────────────────────────┐
│  Файл   Настройки                                    │
├──[📂]──[⚙]───────────────────────────────────────────┤
│                    │                                  │
│  Редактор│Виджеты  │   📈 sin(x) ──────              │
│  ┌──────────────┐  │   📈 cos(x) ──────              │
│  │ Текст...     │  │                                  │
│  │              │  │   (arrastar para zoom)           │
│  └──────────────┘  │                                  │
├────────────────────────────────────────────────────-─┤
│  Стр: 3  Кол: 5  │  Позиция курсора: строка 3 кол 5  │
└──────────────────────────────────────────────────────┘
```

Pressiona **`Ctrl + R`** e manda o print! 🚀
