
# 在 C++ 中与 QML 对象交互

## 简述

所有的 QML 对象类型 - 无论由引擎内部实现还是由第三方源定义，都是 QObject 派生的类型。这意味着，QML 引擎可以使用 Qt 元对象系统动态实例化任何 QML 对象类型并检查创建的对象。

这对于在 C++ 中创建 QML 对象非常有用，无论是显示一个可视化呈现的 QML 对象，还是将非可视 QML 对象数据集成到 C++ 应用程序中。一旦 QML 对象被创建，就可以从 C++ 中检查它，以便读取和写入属性、调用方法、以及接收信号通知。

## 从 C++ 加载 QML 对象

可以使用 QQuickView 或 QQmlComponent 来加载 QML 文档。QQmlComponent 将 QML 文档加载为一个 C++ 对象，然后可以从 C++ 代码中修改该对象。QQuickView 也做到了这一点，但由于 QQuickView 是一个基于 QWindow 的派生类，加载的对象也将被渲染至可视化显示，QQuickView 通常用于将一个可视化的 QML 对象集成到应用程序的用户界面中。

例如，有一个 QML 文件，如下所示：
```
// main.qml
import QtQuick 2.3

Item {
    width: 100; height: 100
}
```

可以使用 QQmlComponent 或 QQuickView 的 C++ 代码加载该 QML 文档。当使用 QQmlComponent 时，需要调用 QQmlComponent::create() 来创建组件的新实例：
```
// 使用 QQmlComponent
QQmlEngine engine;
QQmlComponent component(&engine, QUrl("qrc:/main.qml"));
QObject *object = component.create();
//...
delete object;
```

而 QQuickView 会自动创建组件的实例，该实例可以通过 QQuickView::rootObject() 来访问：

```
// 使用 QQuickView
QQuickView view;
view.setSource(QUrl("qrc:/main.qml"));
view.show();
QObject *object = view.rootObject();
```

实例（object）创建后，就可以使用 QObject::setProperty() 或者 QQmlProperty 来修改其属性：

```
object->setProperty("width", 300);
QQmlProperty(object, "width").write(300);
```

或者，将 object 转换为其实际类型，并使用编译时安全性调用方法。在这种情况下，main.qml 的基本对象是一个 Item，由 QQuickItem 类定义：

```
QQuickItem *item = qobject_cast<QQuickItem*>(object);
item->setWidth(300);
```

## 根据 objectName 访问加载的 QML 对象

QML 组件实质上是具有子对象的对象树，子对象有兄弟，也有孩子。可以使用 QObject::objectName 属性和 QObject::findChild() 来定位 QML 组件的子对象。

例如，如果 QML 中的根 Item 有一个 Rectangle 子项：

```
// main.qml
import QtQuick 2.3

Item {
    width: 100; height: 100

    Rectangle {
        anchors.fill: parent
        objectName: "rect"
    }
}
```

可以通过这样来定位孩子：

```
QObject *rect = object->findChild<QObject*>("rect");
if (rect)
    rect->setProperty("color", "red");
```

注意： 一个对象可能有多个具有相同 objectName 的子项，这种情况下，QObject::findChildren() 可用于查找具有匹配 objectName 的所有子项。

警告： 虽然可以使用 C++ 深入对象树中访问和操作 QML 对象，但建议不要在应用程序测试和原型设计之外采用此方法。QML 和 C++ 集成的一个优势是实现 QML 用户界面独立于 C++ 逻辑和数据集后端，如果 C++ 端深入到 QML 组件中直接操作它们，这种策略就会被打破。对于 C++ 实现，最好尽可能少地了解 QML 用户界面实现和 QML 对象树的组成。
## 从 C++ 访问 QML 对象类型的成员
属性

QML 中声明的任何属性都可以从 C++ 中访问。

例如，下面的 QML 声明了一个简单的字符串：

```
// main.qml
import QtQuick 2.3

Item {
    property string hey: "Hello, Qter!"
}
```
在 C++ 中，属性 hey 的值可以使用 QQmlProperty 来设置和读取，也可使用 QObject::setProperty() 和 QObject::property()：

```
QQmlEngine engine;
QQmlComponent component(&engine, QUrl("qrc:/main.qml"));
QObject *object = component.create();

qDebug() << "Property value:" << QQmlProperty::read(object, "hey").toString();
QQmlProperty::write(object, "hey", "Hello, Qt!");

qDebug() << "Property value:" << object->property("hey").toString();
object->setProperty("hey", "Hello, QML!");
qDebug() << "Property value:" << object->property("hey").toString();
```

输出如下：
```
    Property value: “Hello, Qter!”
    Property value: “Hello, Qt!”
    Property value: “Hello, QML!”
```

注意： 应该始终使用 QObject::setProperty()、QQmlProperty 或 QMetaProperty::write() 来改变 QML 的属性值，以确保 QML 引擎感知属性的变化。

例如，有一个自定义类型 PushButton，它有一个 buttonText 属性。在内部，该属性以成员变量 m_buttonText 来反映值，可以像下面这样直接修改成员变量：

```
// 糟糕的代码
QQmlComponent component(engine, "MyButton.qml");
PushButton *button = qobject_cast<PushButton*>(component.create());
button->m_buttonText = "Click me";
```

但这并不是一个好主意。由于值被直接改变，绕过了 Qt 元对象系统，QML 引擎并没有意识到属性的变化。这意味着，绑定到 buttonText 的属性不会被更新，并且 onButtonTextChanged 处理程序也不会被调用。
调用 QML 方法

所有的 QML 方法都被暴露给了 Qt 元对象系统，可以使用 QMetaObject::invokeMethod() 从 C++ 中调用。从 QML 传递的方法参数和返回值在 C++ 中被转换为 QVariant 值。

写一个简单的 QML，并为其添加一个方法：

```
// main.qml
import QtQuick 2.3

Item {
    function myQmlFunction(msg) {
        console.log("Got message:", msg)
        return "Hello, Qter!"
    }
}
```

然后，在 C++ 中使用 QMetaObject::invokeMethod() 进行调用：

```
// main.cpp
QQmlEngine engine;
QQmlComponent component(&engine, QUrl("qrc:/main.qml"));
QObject *object = component.create();

QVariant returnedValue;  // 返回值
QVariant msg = "Hello, QML!";  // 方法参数
// 调用 QML 方法
QMetaObject::invokeMethod(object, "qmlFunction",
                          Q_RETURN_ARG(QVariant, returnedValue),
                          Q_ARG(QVariant, msg));

qDebug() << "QML function returned:" << returnedValue.toString();
delete object;
```

输出如下：
```
    Got message: Hello, QML!
    QML function returned: “Hello, Qter!”
```

注意： QMetaObject::invokeMethod() 的 Q_RETURN_ARG() 和 Q_ARG() 参数必须被指定为 QVariant 类型，因为这是用于 QML 方法参数和返回值的通用数据类型。
## 连接到 QML 信号

所有的 QML 信号在 C++ 中都是可用的，和普通的 Qt C++ 信号一样，可以使用 QObject::connect() 进行连接。反过来，任何 C++ 信号可以由 QML 对象使用信号处理器来接收。
基本类型的信号参数

这里有一个 QML 组件，包含一个名为 qmlSignal 的信号，该信号包含一个 string 类型参数：

```
// main.qml
import QtQuick 2.3

Item {
    id: item
    width: 100; height: 100

    // QML 信号
    signal qmlSignal(string msg)

    MouseArea {
        anchors.fill: parent
        // 点击鼠标，发射信号
        onClicked: item.qmlSignal("Hello, Qter!")
    }
}
```

写一个 C++ 类，并实现一个槽函数，用于接收 QML 发射的信号：

```
// qter.h
#ifndef QTER_H
#define QTER_H

#include <QObject>
#include <qDebug>

class Qter : public QObject
{
    Q_OBJECT

public slots:
    // 槽函数
    void cppSlot(const QString &msg) {
        qDebug() << "Called the C++ slot with message:" << msg;
    }
};

#endif // QTER_H
```

将信号连接至 C++ 对象的槽函数，当发出 qmlSignal 信号时，就会调用：

```
// qter.h
#include <QGuiApplication>
#include <QQuickView>
#include <QQuickItem>
#include "qter.h"

int main(int argc, char *argv[]) {
    QGuiApplication app(argc, argv);

    QQuickView view(QUrl("qrc:/main.qml"));
    QObject *item = view.rootObject();

    Qter qter;
    // 连接信号槽
    QObject::connect(item, SIGNAL(qmlSignal(QString)), &qter, SLOT(cppSlot(QString)));

    view.show();
    return app.exec();
}
```
点击鼠标后，输出如下：
```
    Called the C++ slot with message: “Hello, Qter!”
```
## 对象类型的信号参数

当信号的参数为 QML 对象类型时，应使用 var 作为参数类型，并且在 C++ 中应使用 QVariant 类型接收该值。

例如，将上述示例中的参数改为 QML 对象类型：

```
// main.qml
import QtQuick 2.3

Item {
    id: item
    width: 100; height: 100

    // QML 信号
    signal qmlSignal(var object)

    MouseArea {
        anchors.fill: parent
        // 点击鼠标，发射信号
        onClicked: item.qmlSignal(item)
    }
}
```
要接收该信号，C++ 类中的槽函数的参数应该改为 QVariant 类型：

```
// qter.h
#ifndef QTER_H
#define QTER_H

#include <QObject>
#include <QQuickItem>
#include <qDebug>

class Qter : public QObject
{
    Q_OBJECT

public slots:
    // 槽函数
    void cppSlot(const QVariant &v) {
        qDebug() << "Called the C++ slot with value:" << v;

        QQuickItem *item = qobject_cast<QQuickItem*>(v.value<QObject*>());
        qDebug() << "Item Size:" << item->width() << item->height();
    }
};

#endif // QTER_H
```

当然，连接信号槽的的参数类型也需要修改：

```
QObject::connect(item, SIGNAL(qmlSignal(QVariant)), &qter, SLOT(cppSlot(QVariant)));
```

点击鼠标后，输出如下：
```
    Called the C++ slot with value: QVariant(QObject*, QQuickItem_QML_0(0x1f4b4cb38d0))
    Item Size: 100 100
```
