## 📁 `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.16)
project(Lab2GUI3 VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOMOC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

add_executable(Lab2GUI3
    main.cpp
    mainwindow.h
    mainwindow.cpp
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
#include <QPlainTextEdit>
#include <QSlider>
#include <QScrollBar>
#include <QSpinBox>
#include <QCheckBox>

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);

protected:
    void closeEvent(QCloseEvent *event) override;
    void resizeEvent(QResizeEvent *event) override;

private slots:
    void onCursorChanged();
    void onCheckBoxToggled(bool checked);
    void openSettings();
    void loadAndDraw();

private:
    QPlainTextEdit *textEditor;
    QLabel         *cursorLabel;

    QSlider    *slider;
    QScrollBar *scrollBar;
    QSpinBox   *spinBox;
    QCheckBox  *checkBox;
    QLabel     *statusLbl;

    QLabel          *canvas;
    QVector<double>  m_xData;
    QVector<double>  m_yData;

    void buildUI();
    void buildMenu();
    void drawGraph();
};

#endif
```

---

## 📁 `mainwindow.cpp`

```cpp
#include "mainwindow.h"

#include <QApplication>
#include <QMenuBar>
#include <QToolBar>
#include <QStatusBar>
#include <QSplitter>
#include <QTabWidget>
#include <QGroupBox>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QAction>
#include <QFileDialog>
#include <QFile>
#include <QTextStream>
#include <QMessageBox>
#include <QDialog>
#include <QDialogButtonBox>
#include <QSettings>
#include <QCloseEvent>
#include <QPainter>
#include <cmath>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    setWindowTitle("Lab 2-3: Приложение с GUI");
    setMinimumSize(820, 520);

    buildUI();
    buildMenu();

    connect(textEditor, &QPlainTextEdit::cursorPositionChanged,
            this, &MainWindow::onCursorChanged);
    onCursorChanged();

    connect(slider, QOverload<int>::of(&QSlider::valueChanged),
            scrollBar, &QScrollBar::setValue);
    connect(slider, QOverload<int>::of(&QSlider::valueChanged),
            spinBox, &QSpinBox::setValue);
    connect(spinBox, QOverload<int>::of(&QSpinBox::valueChanged),
            slider, &QSlider::setValue);
    connect(checkBox, &QCheckBox::toggled,
            this, &MainWindow::onCheckBoxToggled);

    QSettings s;
    if (s.value("saveGeometry", false).toBool())
        restoreGeometry(s.value("geo").toByteArray());

    for (int i = 0; i <= 62; i++) {
        double x = i * 0.1;
        m_xData << x;
        m_yData << std::sin(x);
    }
    drawGraph();
}

void MainWindow::buildUI()
{
    QWidget     *central = new QWidget(this);
    QHBoxLayout *root    = new QHBoxLayout(central);
    root->setContentsMargins(6, 6, 6, 6);
    root->setSpacing(6);
    setCentralWidget(central);

    QSplitter *split = new QSplitter(Qt::Horizontal, central);
    root->addWidget(split);

    // ── Abas (esquerda) ──
    QTabWidget *tabs = new QTabWidget;
    tabs->setMinimumWidth(280);
    tabs->setMaximumWidth(360);

    // Aba 1: Editor
    QWidget     *tabEd = new QWidget;
    QVBoxLayout *layEd = new QVBoxLayout(tabEd);
    layEd->setContentsMargins(6, 6, 6, 6);

    QLabel *lblInfo = new QLabel(
        "Введите текст. Позиция курсора — внизу:");
    lblInfo->setStyleSheet("color:#555; font-size:11px;");

    textEditor = new QPlainTextEdit;
    textEditor->setPlaceholderText(
        "Введите текст здесь...\n\n"
        "Двигайте курсор — внизу обновляется строка и столбец.");
    textEditor->setFont(QFont("Monospace", 11));

    layEd->addWidget(lblInfo);
    layEd->addWidget(textEditor);
    tabs->addTab(tabEd, "Редактор");

    // Aba 2: Виджеты
    QWidget     *tabW  = new QWidget;
    QVBoxLayout *layW  = new QVBoxLayout(tabW);
    layW->setContentsMargins(8, 8, 8, 8);
    layW->setSpacing(12);

    QGroupBox   *grp1  = new QGroupBox(
        "1. QSlider - QScrollBar - QSpinBox");
    QVBoxLayout *layG1 = new QVBoxLayout(grp1);

    slider    = new QSlider(Qt::Horizontal);
    scrollBar = new QScrollBar(Qt::Horizontal);
    spinBox   = new QSpinBox;

    slider->setRange(0, 100);
    scrollBar->setRange(0, 100);
    spinBox->setRange(0, 100);
    scrollBar->setFixedHeight(18);

    QHBoxLayout *rowSpin = new QHBoxLayout;
    rowSpin->addWidget(new QLabel("QSpinBox:"));
    rowSpin->addWidget(spinBox);
    rowSpin->addStretch();

    layG1->addWidget(new QLabel("QSlider:"));
    layG1->addWidget(slider);
    layG1->addWidget(new QLabel("QScrollBar:"));
    layG1->addWidget(scrollBar);
    layG1->addLayout(rowSpin);

    QGroupBox   *grp2  = new QGroupBox("2. QCheckBox - QLabel");
    QHBoxLayout *layG2 = new QHBoxLayout(grp2);

    checkBox  = new QCheckBox("Активировать");
    statusLbl = new QLabel("Статус: Выкл");
    statusLbl->setStyleSheet("color:red; font-weight:bold;");

    layG2->addWidget(checkBox);
    layG2->addWidget(statusLbl);
    layG2->addStretch();

    layW->addWidget(grp1);
    layW->addWidget(grp2);
    layW->addStretch();
    tabs->addTab(tabW, "Виджеты");

    split->addWidget(tabs);

    // ── Canvas gráfico (direita) ──
    canvas = new QLabel;
    canvas->setFrameShape(QFrame::Box);
    canvas->setAlignment(Qt::AlignCenter);
    canvas->setText("График появится здесь");
    canvas->setStyleSheet("background:#FAFAFA; color:#999;");
    canvas->setMinimumWidth(380);
    split->addWidget(canvas);

    split->setStretchFactor(0, 1);
    split->setStretchFactor(1, 2);

    cursorLabel = new QLabel("Стр: 1  Кол: 1");
    cursorLabel->setStyleSheet("padding:0 10px; color:#1565C0;");
    statusBar()->addPermanentWidget(cursorLabel);
    statusBar()->showMessage("Готово", 2000);
}

void MainWindow::buildMenu()
{
    QAction *aLoad = new QAction("Загрузить данные...", this);
    QAction *aSet  = new QAction("Настройки...",        this);
    QAction *aExit = new QAction("Выход",               this);

    aLoad->setShortcut(QKeySequence::Open);
    aExit->setShortcut(QKeySequence::Quit);

    QMenu *mFile = menuBar()->addMenu("Файл");
    mFile->addAction(aLoad);
    mFile->addSeparator();
    mFile->addAction(aExit);

    QMenu *mSet = menuBar()->addMenu("Настройки");
    mSet->addAction(aSet);

    QToolBar *tb = addToolBar("Панель");
    tb->setMovable(false);
    tb->addAction(aLoad);
    tb->addSeparator();
    tb->addAction(aSet);

    connect(aLoad, &QAction::triggered, this, &MainWindow::loadAndDraw);
    connect(aSet,  &QAction::triggered, this, &MainWindow::openSettings);
    connect(aExit, &QAction::triggered, this, &QMainWindow::close);
}

void MainWindow::onCursorChanged()
{
    QTextCursor c   = textEditor->textCursor();
    int ln  = c.blockNumber()  + 1;
    int col = c.columnNumber() + 1;
    cursorLabel->setText(
        QString("Стр: %1  Кол: %2").arg(ln).arg(col));
    statusBar()->showMessage(
        QString("Позиция курсора: строка %1, столбец %2")
            .arg(ln).arg(col), 1500);
}

void MainWindow::openSettings()
{
    QDialog dlg(this);
    dlg.setWindowTitle("Настройки");
    dlg.setFixedSize(340, 160);

    auto *lay  = new QVBoxLayout(&dlg);
    auto *grp  = new QGroupBox("Параметры окна");
    auto *glay = new QVBoxLayout(grp);
    auto *chk  = new QCheckBox(
        "Сохранять положение и размер окна");
    auto *hint = new QLabel(
        "При следующем запуске окно откроется\n"
        "на той же позиции и с теми же размерами.");
    hint->setStyleSheet("color:#777; font-size:11px;");
    auto *bbox = new QDialogButtonBox(
        QDialogButtonBox::Ok | QDialogButtonBox::Cancel);

    QSettings s;
    chk->setChecked(s.value("saveGeometry", false).toBool());

    glay->addWidget(chk);
    glay->addWidget(hint);
    lay->addWidget(grp);
    lay->addWidget(bbox);

    connect(bbox, &QDialogButtonBox::accepted,
            &dlg, &QDialog::accept);
    connect(bbox, &QDialogButtonBox::rejected,
            &dlg, &QDialog::reject);

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

void MainWindow::onCheckBoxToggled(bool on)
{
    statusLbl->setText(on ? "Статус: Вкл" : "Статус: Выкл");
    statusLbl->setStyleSheet(
        on ? "color:green; font-weight:bold;"
           : "color:red;   font-weight:bold;");
}

void MainWindow::loadAndDraw()
{
    QString path = QFileDialog::getOpenFileName(
        this, "Загрузить данные", "",
        "Текстовые файлы (*.txt *.csv *.dat);;Все файлы (*)");
    if (path.isEmpty()) return;

    QFile f(path);
    if (!f.open(QIODevice::ReadOnly | QIODevice::Text)) {
        QMessageBox::warning(this, "Ошибка",
            "Не удалось открыть файл:\n" + path);
        return;
    }

    m_xData.clear();
    m_yData.clear();
    QTextStream in(&f);

    while (!in.atEnd()) {
        QString line = in.readLine().trimmed();
        if (line.isEmpty() || line.startsWith('#')) continue;
        line.replace(',', ' ').replace(';', ' ');
        QStringList parts = line.split(' ', Qt::SkipEmptyParts);
        if (parts.size() >= 2) {
            bool ox, oy;
            double x = parts[0].toDouble(&ox);
            double y = parts[1].toDouble(&oy);
            if (ox && oy) { m_xData << x; m_yData << y; }
        }
    }
    f.close();

    if (m_xData.isEmpty()) {
        QMessageBox::warning(this, "Ошибка",
            "Файл не содержит данных.\n\n"
            "Ожидаемый формат:\n"
            "  0.0  0.0\n"
            "  1.0  1.0\n"
            "  2.0  4.0");
        return;
    }

    drawGraph();
    statusBar()->showMessage(
        QString("Загружено %1 точек  |  %2")
            .arg(m_xData.size())
            .arg(path.section('/', -1)), 4000);
}

void MainWindow::drawGraph()
{
    if (m_xData.isEmpty() || canvas->width() < 20) return;

    const int W  = canvas->width()  - 60;
    const int H  = canvas->height() - 50;
    const int ox = 48;
    const int oy = 16;

    QPixmap pix(canvas->size());
    pix.fill(Qt::white);
    QPainter p(&pix);
    p.setRenderHint(QPainter::Antialiasing);

    double xMin = *std::min_element(m_xData.begin(), m_xData.end());
    double xMax = *std::max_element(m_xData.begin(), m_xData.end());
    double yMin = *std::min_element(m_yData.begin(), m_yData.end());
    double yMax = *std::max_element(m_yData.begin(), m_yData.end());
    if (xMax == xMin) xMax = xMin + 1;
    if (yMax == yMin) yMax = yMin + 1;

    auto tx = [&](double x) -> int {
        return ox + (int)((x - xMin) / (xMax - xMin) * W);
    };
    auto ty = [&](double y) -> int {
        return oy + H - (int)((y - yMin) / (yMax - yMin) * H);
    };

    // Сетка
    p.setPen(QPen(QColor("#EEEEEE"), 1));
    for (int i = 1; i < 5; i++) {
        p.drawLine(ox + W * i / 5, oy,
                   ox + W * i / 5, oy + H);
        p.drawLine(ox, oy + H * i / 5,
                   ox + W, oy + H * i / 5);
    }

    // Оси
    p.setPen(QPen(Qt::black, 1.5));
    p.drawLine(ox, oy, ox, oy + H);
    p.drawLine(ox, oy + H, ox + W, oy + H);

    // Подписи
    p.setFont(QFont("Arial", 8));
    p.setPen(Qt::darkGray);
    p.drawText(ox - 2,      oy + H + 14,
               QString::number(xMin, 'f', 1));
    p.drawText(ox + W - 14, oy + H + 14,
               QString::number(xMax, 'f', 1));
    p.drawText(2, oy + H,
               QString::number(yMin, 'f', 1));
    p.drawText(2, oy + 10,
               QString::number(yMax, 'f', 1));

    // Линия графика
    p.setPen(QPen(QColor("#1565C0"), 2));
    for (int i = 1; i < m_xData.size(); i++)
        p.drawLine(tx(m_xData[i-1]), ty(m_yData[i-1]),
                   tx(m_xData[i]),   ty(m_yData[i]));

    // Точки
    p.setBrush(QColor("#1565C0"));
    p.setPen(QPen(Qt::white, 1));
    for (int i = 0; i < m_xData.size(); i++)
        p.drawEllipse(
            QPoint(tx(m_xData[i]), ty(m_yData[i])), 4, 4);

    // Легенда
    p.setPen(QPen(QColor("#1565C0"), 2));
    p.drawLine(ox + 10, oy + 10, ox + 30, oy + 10);
    p.setPen(Qt::black);
    p.setFont(QFont("Arial", 9));
    p.drawText(ox + 35, oy + 14, "sin(x) / данные из файла");

    canvas->setPixmap(pix);
}

void MainWindow::resizeEvent(QResizeEvent *event)
{
    QMainWindow::resizeEvent(event);
    drawGraph();
}
```

---

## Passos

1. No Qt Creator cria um projeto **Qt Widgets Application**
2. Apaga o `mainwindow.h`, `mainwindow.cpp` e `main.cpp` gerados
3. Cola cada ficheiro acima
4. **`Ctrl + R`** 🚀
