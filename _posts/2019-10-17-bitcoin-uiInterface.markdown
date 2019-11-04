---
layout: post
title:  "bitcoin uiInterface"
date:   2019-10-17 13:00:00 +0800
categories: bitcoin
---

# bitcoin uiInterface

## tox/toxcore.cpp
```
#include <ui_interface.h>

static void friend_request_cb(Tox *tox, const uint8_t *public_key, const uint8_t *message, size_t length, void *user_data)
{
    // ...

    uiInterface.FriendRequest(pubkeyHex, req->msg);
}
```

## ui_interface.h
```
class CClientUIInterface
{
    // ...

    ADD_SIGNALS_DECL_WRAPPER(FriendRequest, void, const std::string&, const std::string&);
};
```

## ui_interface.cpp
```
struct UISignals {
    // ...
    
    boost::signals2::signal<CClientUIInterface::FriendRequestSig> FriendRequest;
} g_ui_signals;

ADD_SIGNALS_IMPL_WRAPPER(FriendRequest);

void CClientUIInterface::FriendRequest(const std::string& pubkey, const std::string& msg) { return g_ui_signals.FriendRequest(pubkey, msg); }
```

## interfaces/node.h
```
class Node
{
    // ...

    //! Register handler for tox friend request.
    using FriendRequestFn = std::function<void(const std::string& pubkey, const std::string& msg)>;
    virtual std::unique_ptr<Handler> handleFriendRequest(FriendRequestFn fn) = 0;
};
```

## interfaces/node.cpp
```
class NodeImpl : public Node
{
    // ...

    std::unique_ptr<Handler> handleFriendRequest(FriendRequestFn fn) override
    {
        return MakeHandler(
            ::uiInterface.FriendRequest_connect([fn](const std::string& pubkey, const std::string& msg) {
                fn(pubkey, msg);
            }));
    }
};
```

## qt/clientmode.h
```
class ClientModel : public QObject
{
    // ...

Q_SIGNALS:
    void friendRequest(const QString& pubkey, const QString& msg);

public Q_SLOTS:
    void getFriendRequest(const QString& pubkey, const QString& msg);

private:
    std::unique_ptr<interfaces::Handler> m_handler_friend_request;
};
```

## qt/clientmode.cpp
```
void ClientModel::getFriendRequest(const QString& pubkey, const QString& msg)
{
    Q_EMIT friendRequest(pubkey, msg);
}

static void FriendRequest(ClientModel* clientModel, const std::string& pubkey, const std::string& msg)
{
    bool invoked = QMetaObject::invokeMethod(clientModel, "getFriendRequest", Qt::QueuedConnection,
                                    Q_ARG(QString, QString::fromStdString(pubkey)),
                                    Q_ARG(QString, QString::fromStdString(msg)));
    assert(invoked);
}

void ClientModel::subscribeToCoreSignals()
{
    // ...

    m_handler_friend_request = m_node.handleFriendRequest(std::bind(FriendRequest, this, std::placeholders::_1, std::placeholders::_2));
}

void ClientModel::unsubscribeFromCoreSignals()
{
    // ...

    m_handler_friend_request->disconnect();
}
``` 

## qt/managementpage.h
```
class ManagementPage : public QWidget
{
    // ...

private Q_SLOTS:
    void onFriendRequest(const QString& pubkey, const QString& msg);
};
```

## qt/managementpage.cpp
```
void ManagementPage::setClientModel(ClientModel *_clientModel)
{
    // ...

    if (_clientModel) {
        connect(_clientModel, &ClientModel::friendRequest, this, &ManagementPage::onFriendRequest);
    }
}

void ManagementPage::onFriendRequest(const QString& pubkey, const QString& msg)
{
    // ...
}
```
