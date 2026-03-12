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
