---
layout:     post
title:      Closing an application without killing the main loop
date:       2015-05-04 19:80:49
summary:    How to run the application main loop on the background and then resume it when the user open the app again.
tags: sailfishos mainloop killing close
---

Reference:

* [Pull data in background | SailfishDevel](https://lists.sailfishos.org/pipermail/devel/2015-May/006038.html)
* [QGuiApplication Class | Qt Gui](http://doc.qt.io/qt-5/qguiapplication.html#quitOnLastWindowClosed-prop)
* [QQuickView Class | Qt Quick](http://doc.qt.io/qt-5/qquickview.html)
* [D-Bus Ping Pong example | QT](http://doc.qt.io/qt-5/qtdbus-pingpong-example.html)
* [How to quit custom app programmatically (Qml)? - together.jolla.com](https://together.jolla.com/question/31043/solved-how-to-quit-custom-app-programmatically-qml/#comment-31228)

Thanks [CODeRUS](https://github.com/coderus/) for the help provided in mailing list!

## Summery

The trick is to set `setQuitOnLastWindowClosed` to `false` in order to prevent the main loop from exiting when the gui window is closed. Then, to re open the window when the app is open again dbus can be used. When the app starts it tries to communicate with a dbus object through the `SessionBus`. If the app can communicate it means that there is another instance running, so the new instance has to exit and the other instance has to show its gui. If the app can't performe the DBus communication, it registers himself as a DBus object. 

## Solution

The solution consist of mainly one class and the main function. A DBus class, `MyDbus` is incharge of stablishing the DBus connection, and for answering DBus calls. This class will be the one who will show the `QQuickView` when another instance of the app tries to run.

The example has another class, `MyTimer`, which only function is to log a message every 3 seconds in order to know the main loop is running.

The `main` function does all the initialization and creation of classes.

[Download full source]({{site.baseurl}}/example/harbour-boxapp.zip).

### `main.cpp`

```
int main(int argc, char *argv[]){
    // A QGuiApplication is created
    QScopedPointer<QGuiApplication> app(SailfishApp::application(argc, argv));

    // The DBus object is created
    MyDbus myDbus;

    if(!myDbus.checkDbus()) // Check if SessionBus works
        return 1;

    if(myDbus.isInstanceRunning()){ // Check if there is another instance running
        qDebug() << "instance already running";
        return 0;
    }

    // Register the object for the DBus call
    myDbus.registerService();

    // Prevent main loop from exiting when the gui is closed
    app->setQuitOnLastWindowClosed(false);

    // Create View
    QScopedPointer<QQuickView> view(SailfishApp::createView());

    // Set qml for the view
    view->setSource(SailfishApp::pathTo("qml/harbour-boxapp.qml"));

   // Show the view
    view->showFullScreen();

    // Set view in the DBus object
    myDbus.setView(view.data());

    // Connect Qt.quit() qml to main loop quit
    QObject::connect(view->engine(), SIGNAL(quit()), app.data(), SLOT(quit()));

    MyTimer t; // Loggin stuff
    qDebug() << "Running!";

    // Run main loop
    int ret = app->exec(); 
    qDebug() << "Closes! with code " << ret ;
    return ret;
}
```

### `mydbus.h`

```
#define SERVICE_NAME "harbour.boxapp.client"

class MyDbus : public QObject
{
    Q_OBJECT
public:
    explicit MyDbus(QObject *parent = 0);
    bool checkDbus();
    bool registerService();
    bool isInstanceRunning();
    void setView(QQuickView* view);

signals:

public slots:
    // This public slot is the one that is executed when another instance of the app checks if an instance is running
    // It is responsible for showing the view
    Q_SCRIPTABLE QString ping(void);

protected:
    QQuickView* view;
};
```

### `mydbus.cpp`

```
MyDbus::MyDbus(QObject *parent) :
    QObject(parent) , view(NULL)
{
}

bool MyDbus::checkDbus(){
    if (!QDBusConnection::sessionBus().isConnected()) {
        qDebug() << "Cannot connect to the D-Bus session bus.";
        return false;
    }
    return true;
}

bool MyDbus::registerService(){
    if(!QDBusConnection::sessionBus().registerService(SERVICE_NAME)){
        qDebug() << qPrintable(QDBusConnection::sessionBus().lastError().message());
        return false;
    }

    if(!QDBusConnection::sessionBus().registerObject("/", this, QDBusConnection::ExportAllSlots)){
        qDebug() << qPrintable(QDBusConnection::sessionBus().lastError().message());
        return false;
    }

    return true;
}

bool MyDbus::isInstanceRunning(){
    QDBusInterface iface(SERVICE_NAME, "/", "", QDBusConnection::sessionBus());
    if (iface.isValid()) {
        QDBusReply<QString> reply = iface.call("ping");
        if (reply.isValid()) {
            qDebug() << "Reply was: ", qPrintable(reply.value());
            return true;
        }

        qDebug() << "Call failed: " << qPrintable(reply.error().message());
        return false;
    }else{
        qDebug() << "invalid iface";
        return false;
    }

    qDebug() << qPrintable(QDBusConnection::sessionBus().lastError().message());

    return true;
}

QString MyDbus::ping(void){
    qDebug() << "Me llaman";
    if(view)
        view->showFullScreen();
    qDebug() << "No explote";
    return "existo!";
}

void MyDbus::setView(QQuickView* view){
    this->view = view;
}
```
