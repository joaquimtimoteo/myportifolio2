## Versão Mínima — sem QCustomPlot, sem dependências externas

O gráfico será feito com **QPainter** (já incluído no Qt). Zero downloads extras.

---

## 📁 `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.16)
project(Lab2GUI3 VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

add_executable(Lab2GUI3
    main.cpp
    mainwindow.h
    mainwindow.cpp
    mainwindow.ui
)

target_link_libraries(Lab2GUI3 PRIVATE
    Qt${QT_VERSION_MAJOR}::Widgets)
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

## 📁 `mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QLabel>

QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT
public:
    explicit MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

protected:
    void closeEvent(QCloseEvent *) override;

private slots:
    void onCursorChanged();       // Задание 1
    void openSettings();          // Задание 2
    void onCheckBoxToggled(bool); // Задание 3
    void loadAndDraw();           // Задание 4/5

private:
    Ui::MainWindow *ui;
    QLabel *cursorLabel;

    QVector<double> m_xData;
    QVector<double> m_yData;

    void setupMenu();
    void drawGraph();
};

#endif
```

---

## 📁 `mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QMenuBar>
#include <QToolBar>
#include <QStatusBar>
#include <QFileDialog>
#include <QFile>
#include <QTextStream>
#include <QMessageBox>
#include <QSettings>
#include <QVBoxLayout>
#include <QDialog>
#include <QCheckBox>
#include <QDialogButtonBox>
#include <QPainter>
#include <cmath>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    setWindowTitle("Lab 2-3");
    setMinimumSize(750, 500);

    setupMenu();

    // StatusBar permanente — Задание 1
    cursorLabel = new QLabel("Стр: 1  Кол: 1");
    statusBar()->addPermanentWidget(cursorLabel);
    statusBar()->showMessage("Готово", 2000);

    connect(ui->textEditor, &QPlainTextEdit::cursorPositionChanged,
            this, &MainWindow::onCursorChanged);

    // Задание 3 — виджеты
    connect(ui->slider, QOverload<int>::of(&QSlider::valueChanged),
            ui->scrollBar, &QScrollBar::setValue);
    connect(ui->slider, QOverload<int>::of(&QSlider::valueChanged),
            ui->spinBox,   &QSpinBox::setValue);
    connect(ui->spinBox, QOverload<int>::of(&QSpinBox::valueChanged),
            ui->slider,    &QSlider::setValue);
    connect(ui->checkBox, &QCheckBox::toggled,
            this, &MainWindow::onCheckBoxToggled);

    // Задание 2 — восстановить геометрию
    QSettings s;
    if (s.value("saveGeometry", false).toBool())
        restoreGeometry(s.value("geo").toByteArray());

    // Демо-данные (sin)
    for (int i = 0; i <= 60; i++) {
        double x = i * 0.1;
        m_xData << x;
        m_yData << std::sin(x);
    }
    drawGraph();
}

MainWindow::~MainWindow() { delete ui; }

// ── Меню ──────────────────────────────────────
void MainWindow::setupMenu()
{
    QAction *aLoad = new QAction("Загрузить файл...", this);
    QAction *aSet  = new QAction("Настройки...",      this);
    QAction *aExit = new QAction("Выход",             this);

    QMenu *mFile = menuBar()->addMenu("Файл");
    mFile->addAction(aLoad);
    mFile->addSeparator();
    mFile->addAction(aExit);

    QMenu *mSet = menuBar()->addMenu("Настройки");
    mSet->addAction(aSet);

    QToolBar *tb = addToolBar("Панель");
    tb->addAction(aLoad);
    tb->addAction(aSet);

    connect(aLoad, &QAction::triggered, this, &MainWindow::loadAndDraw);
    connect(aSet,  &QAction::triggered, this, &MainWindow::openSettings);
    connect(aExit, &QAction::triggered, this, &QMainWindow::close);
}

// ── Задание 1: позиция курсора ─────────────────
void MainWindow::onCursorChanged()
{
    QTextCursor c = ui->textEditor->textCursor();
    int ln = c.blockNumber() + 1;
    int col = c.columnNumber() + 1;
    cursorLabel->setText(
        QString("Стр: %1  Кол: %2").arg(ln).arg(col));
    statusBar()->showMessage(
        QString("Курсор: строка %1, столбец %2").arg(ln).arg(col), 1500);
}

// ── Задание 2: настройки (inline QDialog) ──────
void MainWindow::openSettings()
{
    QDialog dlg(this);
    dlg.setWindowTitle("Настройки");
    dlg.setFixedSize(320, 150);

    auto *lay  = new QVBoxLayout(&dlg);
    auto *chk  = new QCheckBox("Сохранять положение и размер окна");
    auto *bbox = new QDialogButtonBox(
        QDialogButtonBox::Ok | QDialogButtonBox::Cancel);

    QSettings s;
    chk->setChecked(s.value("saveGeometry", false).toBool());

    lay->addWidget(chk);
    lay->addWidget(bbox);

    connect(bbox, &QDialogButtonBox::accepted, &dlg, &QDialog::accept);
    connect(bbox, &QDialogButtonBox::rejected, &dlg, &QDialog::reject);

    if (dlg.exec() == QDialog::Accepted) {
        s.setValue("saveGeometry", chk->isChecked());
        statusBar()->showMessage("Настройки сохранены", 2000);
    }
}

void MainWindow::closeEvent(QCloseEvent *e)
{
    QSettings s;
    if (s.value("saveGeometry", false).toBool())
        s.setValue("geo", saveGeometry());
    e->accept();
}

// ── Задание 3: CheckBox → Label ────────────────
void MainWindow::onCheckBoxToggled(bool on)
{
    ui->statusLbl->setText(on ? "Статус: Вкл ✓" : "Статус: Выкл");
    ui->statusLbl->setStyleSheet(
        on ? "color:green;font-weight:bold"
           : "color:red;font-weight:bold");
}

// ── Задание 4/5: загрузка файла ────────────────
void MainWindow::loadAndDraw()
{
    QString path = QFileDialog::getOpenFileName(
        this, "Загрузить данные", "",
        "Текстовые файлы (*.txt *.csv *.dat)");
    if (path.isEmpty()) return;

    QFile f(path);
    if (!f.open(QIODevice::ReadOnly | QIODevice::Text)) {
        QMessageBox::warning(this, "Ошибка", "Не удалось открыть файл.");
        return;
    }

    m_xData.clear();
    m_yData.clear();
    QTextStream in(&f);
    while (!in.atEnd()) {
        QString line = in.readLine().trimmed();
        if (line.isEmpty() || line.startsWith('#')) continue;
        line.replace(',', ' ').replace(';', ' ');
        QStringList p = line.split(' ', Qt::SkipEmptyParts);
        if (p.size() >= 2) {
            bool ox, oy;
            double x = p[0].toDouble(&ox);
            double y = p[1].toDouble(&oy);
            if (ox && oy) { m_xData << x; m_yData << y; }
        }
    }
    f.close();

    if (m_xData.isEmpty()) {
        QMessageBox::warning(this, "Ошибка",
            "Файл пуст или формат неверный.\nОжидается: x y");
        return;
    }

    drawGraph();
    statusBar()->showMessage(
        QString("Загружено %1 точек").arg(m_xData.size()), 3000);
}

// ── График через QPainter (без внешних библиотек) ──
void MainWindow::drawGraph()
{
    if (m_xData.isEmpty()) return;

    int W = ui->canvas->width()  - 60;
    int H = ui->canvas->height() - 50;
    if (W < 10 || H < 10) return;

    QPixmap pix(ui->canvas->size());
    pix.fill(Qt::white);
    QPainter p(&pix);
    p.setRenderHint(QPainter::Antialiasing);

    // Отступы
    int ox = 40, oy = 20;

    // Найти диапазон
    double xMin = *std::min_element(m_xData.begin(), m_xData.end());
    double xMax = *std::max_element(m_xData.begin(), m_xData.end());
    double yMin = *std::min_element(m_yData.begin(), m_yData.end());
    double yMax = *std::max_element(m_yData.begin(), m_yData.end());

    if (xMax == xMin) xMax = xMin + 1;
    if (yMax == yMin) yMax = yMin + 1;

    // Лямбда: данные → экранные координаты
    auto tx = [&](double x) {
        return ox + (int)((x - xMin) / (xMax - xMin) * W);
    };
    auto ty = [&](double y) {
        return oy + H - (int)((y - yMin) / (yMax - yMin) * H);
    };

    // Оси
    p.setPen(QPen(Qt::black, 1));
    p.drawLine(ox, oy, ox, oy + H);          // Y
    p.drawLine(ox, oy + H, ox + W, oy + H);  // X

    // Подписи осей
    p.setFont(QFont("Arial", 8));
    p.drawText(ox - 35, oy + H + 15,
        QString::number(xMin, 'f', 1));
    p.drawText(ox + W - 15, oy + H + 15,
        QString::number(xMax, 'f', 1));
    p.drawText(2, oy + H,
        QString::number(yMin, 'f', 1));
    p.drawText(2, oy + 8,
        QString::number(yMax, 'f', 1));

    // Линия графика
    p.setPen(QPen(QColor("#1565C0"), 2));
    for (int i = 1; i < m_xData.size(); i++) {
        p.drawLine(tx(m_xData[i-1]), ty(m_yData[i-1]),
                   tx(m_xData[i]),   ty(m_yData[i]));
    }

    // Точки
    p.setBrush(QColor("#1565C0"));
    for (int i = 0; i < m_xData.size(); i++)
        p.drawEllipse(tx(m_xData[i]) - 3,
                      ty(m_yData[i]) - 3, 6, 6);

    ui->canvas->setPixmap(pix);
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
   <rect><x>0</x><y>0</y><width>860</width><height>560</height></rect>
  </property>
  <widget class="QWidget" name="centralwidget">
   <layout class="QHBoxLayout">
    <item>
     <!-- ESQUERDA: abas -->
     <widget class="QTabWidget" name="tabWidget">
      <property name="maximumWidth"><number>340</number></property>

      <!-- Aba 1: Editor -->
      <widget class="QWidget" name="tab1">
       <attribute name="title"><string>Редактор</string></attribute>
       <layout class="QVBoxLayout">
        <item>
         <widget class="QPlainTextEdit" name="textEditor">
          <property name="placeholderText">
           <string>Введите текст...
Позиция курсора — в строке состояния.</string>
          </property>
         </widget>
        </item>
       </layout>
      </widget>

      <!-- Aba 2: Виджеты -->
      <widget class="QWidget" name="tab2">
       <attribute name="title"><string>Виджеты</string></attribute>
       <layout class="QVBoxLayout">
        <item>
         <widget class="QGroupBox">
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
             <item><widget class="QSpinBox" name="spinBox"><property name="maximum"><number>100</number></property></widget></item>
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
           <item><widget class="QCheckBox" name="checkBox"><property name="text"><string>Активировать</string></property></widget></item>
           <item><widget class="QLabel" name="statusLbl"><property name="text"><string>Статус: Выкл</string></property><property name="styleSheet"><string>color:red;font-weight:bold</string></property></widget></item>
           <item><spacer><property name="orientation"><enum>Qt::Horizontal</enum></property></spacer></item>
          </layout>
         </widget>
        </item>
        <item><spacer><property name="orientation"><enum>Qt::Vertical</enum></property></spacer></item>
       </layout>
      </widget>

     </widget>
    </item>

    <item>
     <!-- DIREITA: gráfico -->
     <widget class="QLabel" name="canvas">
      <property name="frameShape"><enum>QFrame::Box</enum></property>
      <property name="alignment"><set>Qt::AlignCenter</set></property>
      <property name="text"><string>График появится здесь</string></property>
     </widget>
    </item>

   </layout>
  </widget>
 </widget>
</ui>
```

---

## Resultado

```
┌──────────────────────────────────────────────────┐
│  Файл   Настройки                                │
├──[Загрузить]──[Настройки]────────────────────────┤
│ Редактор│Виджеты │                               │
│ ┌──────────────┐ │   📈 График (QPainter)        │
│ │ текст...     │ │                               │
│ │              │ │   ● ─── ● ─── ●              │
│ └──────────────┘ │                               │
├──────────────────────────────────────────────────┤
│ Курсор: строка 2, столбец 5  │  Стр: 2  Кол: 5  │
└──────────────────────────────────────────────────┘
```

**Vantagens desta versão:**
- Zero dependências externas
- Um único ficheiro `.cpp` para tudo
- O diálogo de Settings está inline (sem ficheiro separado)
- Gráfico com `QPainter` nativo do Qt
- Compila imediatamente com **`Ctrl + R`** 🚀
