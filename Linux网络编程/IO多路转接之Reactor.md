# IO多路转接之Reactor
epoll的工作模式有两种，一种默认`LT`工作模式，另一种是`ET`模式。关于epoll的`LT`工作模式我们已经写过了。接下来我们写一份基于`ET`模式下的Reator，处理所有的IO。

> Reactor = 如何正确的处理IO+协议定制+业务逻辑处理
>

下面我们写一个Reactor，它是一个半同步半异步IO，Reactor是在Liunx网络中，最常用，最频繁的一种网络IO设计模式！

# 准备工作
## Logger.hpp
```cpp
#ifndef __LOGGER_HPP__
#define __LOGGER_HPP__

#include <iostream>
#include <string>
#include <ctime>
#include <unistd.h>
#include <memory>
#include <sstream>
#include <filesystem>
#include <fstream>

#include "lockGuard.hpp"
// 日志等级

enum class LogLevel
{
    DEBUG,
    INFO,
    WARNING,
    ERROR,
    FATAL
};

std::string Level2String(LogLevel level)
{
    switch (level)
    {
    case LogLevel::DEBUG:
        return "Debug";
    case LogLevel::INFO:
        return "Info";
    case LogLevel::WARNING:
        return "Warning";
    case LogLevel::ERROR:
        return "Error";
    case LogLevel::FATAL:
        return "Fatal";
    default:
        return "Unknown";
    }
}

std::string getCurrentTime()
{
    // 获取当前时间戳
    time_t currtime = time(nullptr);

    // 转换时间
    struct tm t;
    localtime_r(&currtime, &t);

    char timebuffer[64];

    snprintf(timebuffer, sizeof(timebuffer), "%4d-%02d-%02d %02d:%02d:%02d",
             t.tm_year + 1900,
             t.tm_mon + 1,
             t.tm_mday,
             t.tm_hour,
             t.tm_min,
             t.tm_sec);
    return timebuffer;
}

class LogStrategy
{
public:
    virtual ~LogStrategy() = default;
    virtual void SyncLog(const std::string &logmessage) = 0; // 刷新日志
};

class ConsoleLogStrategy : public LogStrategy
{
public:
    ~ConsoleLogStrategy()
    {
    }
    void SyncLog(const std::string &logmessage) override
    {
        {
            LockGuard lockGuard(&_lock);
            std::cout << logmessage << std::endl;
        }
    }

private:
    Mutex _lock;
};

const std::string logdefaultdir = "./log";
const static std::string logfilename = "test.log";

class FileLogStrategy : public LogStrategy
{
public:
    FileLogStrategy(const std::string &dir = logdefaultdir, const std::string &logfilename = logfilename)
        : _dir_path_name(dir), _file_name(logfilename)
    {
        {
            LockGuard lockGuard(&_lock);
            if (std::filesystem::exists(_dir_path_name))
            {
                return;
            }
            try
            {
                std::filesystem::create_directories(_dir_path_name);
            }
            catch (const std::filesystem::filesystem_error &e)
            {
                std::cerr << e.what() << "\r\n";
            }
        }
    }
    void SyncLog(const std::string &logmessage) override
    {
        {
            LockGuard lockGuard(&_lock);
            std::string target = _dir_path_name;
            target += "/";
            target += _file_name;
            std::ofstream out(target.c_str(), std::ios::app);
            if (!out.is_open())
            {
                std::cerr << "Failed to open log file: " << target << "\n";
                return;
            }
            out << logmessage << "\n";
            out.close();
        }
    }
    ~FileLogStrategy() {}

private:
    std::string _dir_path_name;
    std::string _file_name;
    Mutex _lock;
};

class Logger
{
public:
    Logger()
    {
        EnableConsoleLogStrategy();
    }

    void EnableConsoleLogStrategy()
    {
        _strategy = std::make_unique<ConsoleLogStrategy>();
    }

    void EnableFileLogStrategy()
    {
        _strategy = std::make_unique<FileLogStrategy>();
    }

    // 一条消息
    class LogMessage
    {
    public:
        LogMessage(LogLevel level, std::string filename, int line, Logger &logger)
            : _curr_time(getCurrentTime()), _level(level), _pid(getpid()), _filename(filename), _line(line), _logger(logger)
        {
            std::stringstream ss;
            ss << "[" << _curr_time << "] "
               << "[" << Level2String(_level) << "] "
               << "[" << _pid << "] "
               << "[" << _filename << "] "
               << "[" << _line << "] "
               << ": ";
            _loginfo = ss.str();
        }

        template <typename T>
        LogMessage &operator<<(const T &info)
        {
            std::stringstream ss;
            ss << info;
            _loginfo += ss.str();
            return *this;
        }

        ~LogMessage()
        {
            // 在析构函数中刷新日志
            if (_logger._strategy)
            {
                _logger._strategy->SyncLog(_loginfo);
            }
        }

    private:
        std::string _curr_time; // 当前时间
        LogLevel _level;        // 告警级别
        pid_t _pid;             // 进程pid
        std::string _filename;  // 文件名字
        int _line;              // 行号

        std::string _loginfo; // 信息主体
        Logger &_logger;      // 提供刷新策略的具体做法
    };

    LogMessage operator()(LogLevel level, std::string filename, int line)
    {
        return LogMessage(level, filename, line, *this);
    }

    ~Logger() {}

private:
    std::unique_ptr<LogStrategy> _strategy;
};

Logger logger; // 全局日志对象

#define LOG(level) logger(level, __FILE__, __LINE__)
#define EnableConsoleLog() logger.EnableConsoleLogStrategy()
#define EnableFileLog() logger.EnableFileLogStrategy()

#endif // __LOGGER_HPP__

```

## lockGuard.hpp
```cpp
#ifndef __LOCK_GUARD_HPP__
#define __LOCK_GUARD_HPP__
#include <pthread.h>
class Mutex
{
public:
    Mutex()
    {
        pthread_mutex_init(&_lock, nullptr);
    }
    void Lock()
    {
        pthread_mutex_lock(&_lock);
    }
    void Unlock()
    {
        pthread_mutex_unlock(&_lock);
    }
    pthread_mutex_t *Get()
    {
        return &_lock;
    }
    ~Mutex()
    {
        pthread_mutex_destroy(&_lock);
    }

private:
    pthread_mutex_t _lock;
};

class LockGuard
{
public:
    LockGuard(Mutex *mutex) : _mutex(mutex)
    {
        _mutex->Lock();
    }
    ~LockGuard()
    {
        _mutex->Unlock();
    }

private:
    Mutex *_mutex;
};
#endif
```

## InetAddr.hpp
```cpp
#ifndef INET_ADDR_HPP
#define INET_ADDR_HPP

#include <iostream>
#include <string>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <cstring>

class InetAddr
{
private:
    void Host2Net()
    {
        bzero(&_addr, sizeof(_addr));
        _addr.sin_family = AF_INET; // 使用IPv4地址族
        _addr.sin_port = htons(_port);
        // _addr.sin_addr.s_addr = inet_addr(_ip.c_str()); // 线程不安全

        inet_pton(AF_INET, _ip.c_str(), &(_addr.sin_addr.s_addr)); // 线程安全  
    }
    void Net2Host()
    {
        _port = ntohs(_addr.sin_port);
        // _ip = inet_ntoa(_addr.sin_addr); // 线程不安全
        
        char buffer[64];
        inet_ntop(AF_INET, &(_addr.sin_addr.s_addr), buffer, sizeof(buffer)); // 建议使用
    }

public:
    InetAddr(){}
    InetAddr(struct sockaddr_in &addr)
        : _addr(addr)
    {
        Net2Host(); // 将网络字节序转换为主机字节序
    }

    InetAddr(uint16_t port, const std::string &ip = "0.0.0.0")
        : _port(port), _ip(ip)
    {
        Host2Net(); // 将主机字节序转换为网络字节序
    }

    const std::string &IP() const { return _ip; }
    uint16_t Port() const { return _port; }

    const struct sockaddr *Addr() const
    {
        return reinterpret_cast<const struct sockaddr *>(&_addr);
    }
    struct sockaddr *Addr()
    {
        return reinterpret_cast<struct sockaddr *>(&_addr);
    }
    socklen_t Length() const
    {
        return sizeof(_addr);
    }

    std::string ToString() const
    {
        return _ip + ":" + std::to_string(_port);
    }

    bool operator==(const InetAddr &addr) const
    {
        return (_ip == addr._ip) && (_port == addr._port);
    }

    void Init(const struct sockaddr_in &addr)
    {
        _addr = addr;
        Net2Host();
    }

    ~InetAddr()
    {
    }

private:
    struct sockaddr_in _addr;
    std::string _ip;
    uint16_t _port;
};

#endif // INET_ADDR_HPP
```

## Socket.hpp
```cpp
#ifndef __SOCKET_HPP__
#define __SOCKET_HPP__

#include <sys/types.h>
#include <sys/socket.h>

#include "Logger.hpp"
#include "InetAddr.hpp"

enum
{
    OK,
    CREATE_ERR,
    BIND_ERR,
    LISTEN_ERR,
    ACCEPT_ERR
};

const static int gbacklog = 16;

class Socket
{
public:
    virtual ~Socket() {}
    virtual void CreateSocketOrDie() = 0;
    virtual void BindSocketOrDie(int port) = 0;
    virtual void ListenSocketOrDie(int gbacklog) = 0;
    // virtual std::shared_ptr<Socket> Accept(InetAddr *clientaddr) = 0;
    virtual int Accept(InetAddr *clientaddr, int *error) = 0;
    virtual int SockFd() = 0;
    virtual void Close() = 0;
    virtual ssize_t Recv(std::string *out) = 0;
    virtual ssize_t Send(const std::string &in) = 0;
    virtual bool Connect(InetAddr &peer) = 0;

public:
    void BuildListenSocketMethod(int port)
    {
        CreateSocketOrDie();
        BindSocketOrDie(port);
        ListenSocketOrDie(gbacklog);
    }
    void BuildClientSocketMethod()
    {
        CreateSocketOrDie();
    }
};

class TcpSocket : public Socket
{
public:
    TcpSocket() {}
    TcpSocket(int sockfd) : _sockfd(sockfd) {}

    void CreateSocketOrDie() override
    {
        _sockfd = socket(AF_INET, SOCK_STREAM, 0);
        if (_sockfd < 0)
        {
            LOG(LogLevel::FATAL) << "create socket error!";
            exit(CREATE_ERR);
        }
        LOG(LogLevel::INFO) << "create socket success! fd: " << _sockfd;

        // 设置地址复用
        int opt = 1;
        setsockopt(_sockfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    }
    void BindSocketOrDie(int port) override
    {
        InetAddr local(port);
        if (bind(_sockfd, local.Addr(), local.Length()))
        {
            LOG(LogLevel::FATAL) << "bind socket error!";
            exit(BIND_ERR);
        }
        LOG(LogLevel::INFO) << "bind socket success!";
    }
    void ListenSocketOrDie(int backlog) override
    {
        if (listen(_sockfd, backlog) != 0)
        {
            LOG(LogLevel::FATAL) << "listen socket error!";
            exit(LISTEN_ERR);
        }
        LOG(LogLevel::INFO) << "listen socket success!";
    }

    int Accept(InetAddr *clientaddr, int *error) override
    {
        struct sockaddr_in peer;
        socklen_t len = sizeof(peer);
        int fd = accept(_sockfd, (struct sockaddr *)&peer, &len);
        if (fd < 0)
        {
            *error = errno;
            LOG(LogLevel::WARNING) << "accept socket error!";
            return -1;
        }
        LOG(LogLevel::INFO) << "accept socket success!";
        clientaddr->Init(peer); // 设置
        return fd;
    }
    int SockFd() override
    {
        return _sockfd;
    }
    void Close() override
    {
        if (_sockfd >= 0)
            close(_sockfd);
    }
    ssize_t Recv(std::string *out) override
    {
        char buffer[1024];
        ssize_t n = recv(_sockfd, buffer, sizeof(buffer) - 1, 0);
        if (n > 0)
        {
            buffer[n] = 0;
            *out += buffer;
        }
        return n;
    }
    ssize_t Send(const std::string &in) override
    {
        return send(_sockfd, in.c_str(), in.size(), 0);
    }
    bool Connect(InetAddr &peer) override
    {
        int n = connect(_sockfd, peer.Addr(), peer.Length());
        if (n >= 0)
            return true;
        else
            return false;
    }

    ~TcpSocket() {}

private:
    int _sockfd;
};

#endif
```

# 计算、解析
## Calculator.hpp
```cpp
#pragma once

#include "Protocol.hpp"

class Calculator
{
public:
    Calculator(){}
    /**
     * code: 0 计算正确
     * code: 1 除零错误
     * code: 2 取模错误
     * code: 3 未支持
     */
    Response Exec(Request &req)
    {
        Response resp;
        switch (req.GetOper())
        {
        case '+':
            resp.SetResult(req.GetX() + req.GetY());
            break;
        case '-':
            resp.SetResult(req.GetX() - req.GetY());
            break;
        case '*':
            resp.SetResult(req.GetX() * req.GetY());
            break;
        case '/':
        {
            if (req.GetY() == 0)
            {
                resp.SetCode(1);
            }
            else
            {
                resp.SetResult(req.GetX() / req.GetY());
            }
        }
        break;
        case '%':
        {
            if (req.GetY() == 0)
            {
                resp.SetCode(2);
            }
            else
            {
                resp.SetResult(req.GetX() % req.GetY());
            }
        }
        break;
        case '^':
            resp.SetResult(req.GetX() ^ req.GetY());
            break;
        case '|':
            resp.SetResult(req.GetX() | req.GetY());
            break;
        case '&':
            resp.SetResult(req.GetX() & req.GetY());
            break;
        default:
            resp.SetCode(3);
            break;
        }
        return resp;
    }
    ~Calculator() {}
};
```

## Protocol.hpp
```cpp
#pragma once

#include <iostream>
#include <string>
#include <jsoncpp/json/json.h>

static const std::string sep = "\r\n";

class Request
{
public:
    Request() : _x(0), _y(0), _oper(0)
    {
    }

    bool Serialize(std::string *out)
    {
        Json::Value root;
        root["x"] = _x;
        root["y"] = _y;
        root["oper"] = _oper;

        Json::StyledWriter writer;
        *out = writer.write(root);

        if (out->empty())
            return false;
        return true;
    }

    bool Deserialize(std::string &in)
    {
        Json::Reader reader;
        Json::Value root;
        if (!reader.parse(in, root))
            return false;

        _x = root["x"].asInt();
        _y = root["y"].asInt();
        _oper = root["oper"].asInt();
        return true;
    }

    int GetX()
    {
        return _x;
    }

    int GetY()
    {
        return _y;
    }

    char GetOper()
    {
        return _oper;
    }

    void SetX(int x)
    {
        _x = x;
    }

    void SetY(int y)
    {
        _y = y;
    }

    void SetOper(char oper)
    {
        _oper = oper;
    }

    ~Request() {}

private:
    int _x;
    int _y;
    char _oper;
};

class Response
{
public:
    Response() : _result(0), _code(0)
    {
    }

    bool Serialize(std::string *out)
    {
        Json::Value root;
        root["result"] = _result;
        root["code"] = _code;

        Json::StyledWriter writer;
        *out = writer.write(root);

        if (out->empty())
            return false;
        return true;
    }

    bool Deserialize(std::string &in)
    {
        Json::Reader reader;
        Json::Value root;
        if (!reader.parse(in, root))
            return false;

        _result = root["result"].asInt();
        _code = root["code"].asInt();
        return true;
    }

    void Print()
    {
        std::cout << "result: " << _result << " [" << _code << "]" << std::endl;
    }

    int GetResult()
    {
        return _result;
    }

    int GetCode()
    {
        return _code;
    }

    void SetResult(int result)
    {
        _result = result;
    }

    void SetCode(int code)
    {
        _code = code;
    }

    ~Response() {}

private:
    int _result;
    int _code;
};


class ProtoCol
{
public:
    static std::string Package(const std::string &jsonstr)
    {
        if (jsonstr.empty())
            return std::string();

        std::string json_len = std::to_string(jsonstr.size());

        return json_len + sep + jsonstr + sep; // 有效载荷长度\r\n有效载荷内容\r\n
    }

    /**
     *  返回值说明：
     *              0:表示没有内容
     *             -1:表示错误字符
     *             <0: 表示解包后的字符串的长度
     */
    static int Unpack(std::string &origin_str, std::string *package)
    {
        if (!package)
            return 0;
        auto pos = origin_str.find(sep); // 查找\r\n
        if (pos == std::string::npos)
            return 0;

        std::string len_str = origin_str.substr(0, pos); // 截取有效载荷字符串
        if (!DigitSafeCheck(len_str))
            return -1;

        int digit_len = std::stoi(len_str);                                    // 获取有效载荷长度
        int target_len = len_str.size() + sep.size() + digit_len + sep.size(); // eg:  3 \r\n *** \r\n

        if (origin_str.size() < target_len)
            return 0;

        *package = origin_str.substr(pos + sep.size(), digit_len); // 把有效载荷长度带出去
        origin_str.erase(0, target_len);                           // 删除前面的数据

        return package->size();
    }

private:
    static bool DigitSafeCheck(const std::string str)
    {
        for (const auto &ch : str)
        {
            if (!(ch >= '0' && ch <= '9'))
                return false;
        }
        return true;
    }
};
```

## Parser.hpp
```cpp
#pragma once

#include <functional>

#include "Parser.hpp"
#include "Calculator.hpp"
#include "Protocol.hpp"
#include "Logger.hpp"

using handler_t = std::function<Response(Request &req)>;

class Parser
{
public:
    Parser(handler_t handler) : _handler(handler)
    {
    }

    std::string Parse(std::string &inbuffer)
    {
        LOG(LogLevel::DEBUG) << "inbuffer: \r\n"
                             << inbuffer;

        std::string send_str;
        for (;;) // 获取到的数据不一定是全部的，需要循环获取, 当获取完一个整个报文后回掉回去进行发送
        {
            std::string jsonstr;
            // 解析报文
            int n = ProtoCol::Unpack(inbuffer, &jsonstr);
            if (n < 0)
                break; // 解包错误
            else if (n == 0)
                break; // 已经处理完毕了
            else 
            {
                LOG(LogLevel::DEBUG) << "jsonstr: \r\n"
                                     << jsonstr;

                // 反序列化
                Request req;
                if (!req.Deserialize(jsonstr))
                {
                    return std::string();
                }

                Response resp = _handler(req); // 回掉函数到上层处理业务

                // 序列化
                std::string resp_json;
                if (!resp.Serialize(&resp_json))
                {
                    return std::string();
                }

                // 打包
                send_str += ProtoCol::Package(resp_json);
            }
        }
        return send_str; // 回掉回去然后进行发送数据
    }

private:
    handler_t _handler;
};

```

# Reactor实现
## Util.hpp
```cpp
#pragma once

#include <fcntl.h>
#include <string>
#include <sys/epoll.h>

void SetNonBlock(int fd)
{
    int fl = fcntl(fd, F_GETFL);
    if (fl < 0)
    {
        LOG(LogLevel::WARNING) << "设置非阻塞失败, fd: " << fd;
        return;
    }
    LOG(LogLevel::INFO) << "设置非阻塞成功, fd: " << fd;
    fcntl(fd, F_SETFL, fl | O_NONBLOCK);
}

std::string Event2String(uint32_t events)
{
    std::string s;
    if (events & EPOLLIN)
    {
        s = "EPOLLIN";
    }
    if (events & EPOLLOUT)
    {
        s += "|EPOLLOUT";
    }
    if (events & EPOLLET)
    {
        s += "|EPOLLET";
    }
    if (events & EPOLLHUP)
    {
        s += "|EPOLLHUP";
    }
    if (events & EPOLLERR)
    {
        s += "|EPOLLERR";
    }
    return s;
}

```

## Connection.hpp
```cpp
#pragma once

#include <functional>
#include <sys/epoll.h>
#include "Logger.hpp"
#include "Reactor.hpp"
#include "InetAddr.hpp"

class Reactor;

using callback_t = std::function<std::string(std::string &inbuffer)>;

class Connection
{
public:
    Connection() : _events(0), _owner(nullptr)
    {
    }
    virtual void Recver() = 0;
    virtual void Sender() = 0;
    virtual void Excepter() = 0;
    virtual ~Connection() {}
    int Sockfd() { return _sockfd; }
    void SetSocketfd(int sockfd) { _sockfd = sockfd; }
    void SetEvents(uint32_t events) { _events = events; }
    uint32_t Events() { return _events; }
    void SetAddr(const InetAddr &addr) { _peer = addr; }
    Reactor *Owner() { return _owner; }
    void SetOwner(Reactor *r) { _owner = r; }
    void SetCallBack(callback_t cb) { _cb = cb; }

protected:
    int _sockfd;            // 连接的socketfd
    uint32_t _events;       // 连接的事件
    InetAddr _peer;         // 连接的地址
    std::string _inbuffer;  // 连接的输入缓冲区
    std::string _outbuffer; // 连接的输出缓冲区

    Reactor *_owner; // 方便添加使用Reactor方法

    callback_t _cb;
};
```

### Listener.hpp
```cpp
#pragma once

#include "Connection.hpp"
#include "Logger.hpp"
#include "Socket.hpp"
#include "Util.hpp"
#include "Channel.hpp"

class Listener : public Connection
{
public:
    Listener(uint16_t port)
        : _listensockfd(std::make_unique<TcpSocket>()), _port(port)
    {
        _listensockfd->BuildListenSocketMethod(_port);
        _sockfd = _listensockfd->SockFd();
        _events = EPOLLIN | EPOLLET;
        SetNonBlock(_sockfd);
    }

    void Recver() override
    {
        for (;;)
        {
            InetAddr clientaddr;
            int error = 0;
            int sockfd = _listensockfd->Accept(&clientaddr, &error);
            if (sockfd < 0)
            {
                if (error == EAGAIN)
                    break;
                else if (error == EINTR)
                    continue;
                else
                    break;
            }

            SetNonBlock(sockfd);
            // 构建新的连接
            std::shared_ptr<Connection> conn = std::make_shared<Channel>(sockfd, clientaddr);
            conn->SetCallBack(_cb);
            // 添加到Reactor中
            Owner()->AddConnection(conn);
        }
    }

    void Sender() override
    {
        LOG(LogLevel::DEBUG) << "Listener Sender";
    }
    void Excepter() override
    {
        LOG(LogLevel::DEBUG) << "Listener Excepter";
    }

    ~Listener()
    {
    }

private:
    std::unique_ptr<Socket> _listensockfd;
    uint16_t _port;
};

```

### channel.hpp
```cpp
#pragma once
#include "Epoller.hpp"
#include "Connection.hpp"

class Channel : public Connection
{
    static const int buffersize = 1024;

public:
    Channel(int sockfd, InetAddr &addr)
    {
        _sockfd = sockfd;
        _peer = addr;
        _events = EPOLLIN | EPOLLET;
    }

    void Recver() override
    {
        for (;;)
        {
            char buffer[buffersize];
            ssize_t n = recv(_sockfd, buffer, sizeof(buffer) - 1, 0);
            if (n > 0)
            {
                buffer[n] = 0;
                _inbuffer += buffer;
            }
            else if (n == 0)
            {
                LOG(LogLevel::INFO) << "client quit, client is : " << _peer.ToString();
                Excepter();
                return; 
            }
            else
            {
                if (errno == EAGAIN)
                {
                    break;
                }
                else if (errno == EINTR)
                {
                    continue;
                }
                else
                {
                    LOG(LogLevel::INFO) << "recv error, client is : " << _peer.ToString();
                    Excepter();
                    return; 
                }
            }
        }

        // 一定把数据全部读完成了
        _outbuffer = _cb(_inbuffer);
        std::cout << "_outbuffer: " << _outbuffer << std::endl;

        // if (!_outbuffer.empty())
        //     Owner()->EnableReadWrite(_sockfd, true, true);

        // 最佳实践 -- 直接发送
        if(!_outbuffer.empty())
            Sender();
    }

    void Sender() override
    {
        for (;;)
        {
            ssize_t n = send(_sockfd, _outbuffer.c_str(), _outbuffer.size(), 0);
            if (n > 0)
            {
                _outbuffer.erase(0, n);
                if (_outbuffer.empty())
                    break;
            }
            else if (n == 0)
                break;
            else if (n < 0)
            {
                if (errno == EAGAIN)
                    break;
                else if (errno == EINTR)
                    continue;
                else
                {
                    Excepter();
                    return;
                }
            }
        }
        // 1. 发送完成 2. 缓冲区写满
        if (!_outbuffer.empty())
            Owner()->EnableReadWrite(_sockfd, true, true);
        else
            Owner()->EnableReadWrite(_sockfd, true, false);
    }

    void Excepter() override
    {
        Owner()->DelConnection(_sockfd);
    }

    ~Channel() {}
};

```

## Epoller.hpp
```cpp
#pragma once

#include <sys/epoll.h>
#include "Logger.hpp"


class Epoller
{
private:
    int OperEventHelper(int sockfd, uint32_t events, int op)
    {
        struct epoll_event ev;
        ev.data.fd = sockfd;
        ev.events = events;
        return epoll_ctl(_epfd, op, sockfd, &ev);
    }

public:
    Epoller()
    {
        _epfd = epoll_create(128);
        if(_epfd < 0)
        {
            LOG(LogLevel::FATAL) << "epoll create fatal!";
            exit(1);
        }
        LOG(LogLevel::INFO) << "create epoll fd success, epfd: " << _epfd;
    }

    void AddEvent(int sockfd, uint32_t events)
    {
        int n = OperEventHelper(sockfd, events, EPOLL_CTL_ADD);
        if (n != 0)
        {
            LOG(LogLevel::INFO) << "add: " << sockfd << " events: "
                                << Event2String(events) << " to epoller failed";
            return;
        }
        LOG(LogLevel::INFO) << "add: " << sockfd << " events: "
                            << Event2String(events) << " to epoller success";
    }
    void DelEvent(int sockfd)
    {
        int n = epoll_ctl(_epfd, EPOLL_CTL_DEL, sockfd, nullptr);
        if (n != 0)
        {
            LOG(LogLevel::INFO) << "Del: " << sockfd << " from epoller failed";
            return;
        }
        LOG(LogLevel::INFO) << "Del: " << sockfd << " from epoller success";
    }

    void ModEvent(int sockfd, uint32_t events)
    {
        int n = OperEventHelper(sockfd, events, EPOLL_CTL_MOD);
        if (n != 0)
        {
            LOG(LogLevel::INFO) << "Mod: " << sockfd << " events: "
                                << Event2String(events) << " to epoller failed";
            return;
        }
        LOG(LogLevel::INFO) << "Mod: " << sockfd << " events: "
                            << Event2String(events) << " to epoller success";
    }

    int Wait(struct epoll_event revs[], int num, int timeout)
    {
        int n = epoll_wait(_epfd, revs, num, timeout);
        if(n < 0)
        {
            LOG(LogLevel::WARNING) << "wait error!";
            return -1;
        }
        return n;
    }

    ~Epoller()
    {
        if(_epfd >= 0)
        {
            close(_epfd);
        }
    }
private:
    int _epfd;
};


```

## Reactor.hpp
```cpp
#pragma once

#include <unordered_map>
#include <memory>
#include <time.h>

#include "Util.hpp"
#include "Epoller.hpp"
#include "Connection.hpp"

class Reactor
{
    const static int size = 128;

private:
    bool IsExist(std::shared_ptr<Connection> &conn)
    {
        auto it = _connections.find(conn->Sockfd());
        return it != _connections.end();
    }

    bool IsExist(int sockfd)
    {
        auto it = _connections.find(sockfd);
        return it != _connections.end();
    }

public:
    Reactor() : _epoller(std::make_unique<Epoller>()) {}

    void AddConnection(std::shared_ptr<Connection> &conn)
    {
        if (IsExist(conn))
        {
            LOG(LogLevel::WARNING) << conn->Sockfd() << "conn in Reactor!";
            return;
        }
        conn->SetOwner(this); // 回指当前reactor
        _connections.insert(std::make_pair(conn->Sockfd(), conn));
        _epoller->AddEvent(conn->Sockfd(), conn->Events());
        LOG(LogLevel::INFO) << conn->Sockfd() << " conn add to Reactor";
    }

    void EnableReadWrite(int sockfd, bool enableread, bool enablewrite)
    {
        if (!IsExist(sockfd))
        {
            LOG(LogLevel::WARNING) << sockfd << " conn not in Reactor[EnableReadWrite]";
            return;
        }
        // 1. 修改connection对象
        uint32_t events = (enableread ? EPOLLIN : 0) | (enablewrite ? EPOLLOUT : 0) | EPOLLET;
        _connections[sockfd]->SetEvents(events);

        // 2. 写入到内核
        _epoller->ModEvent(sockfd, events);
        // _epoller->ModEvent(_connections[sockfd]->Sockfd(), _connections[sockfd]->Events());
    }

    void DelConnection(int sockfd)
    {
        if (!IsExist(sockfd))
        {
            LOG(LogLevel::WARNING) << sockfd << " conn not in Reactor[delete]!";
            return;
        }
        // 1. 从epoll中删除
        _epoller->DelEvent(sockfd);
        // 2. 从map中删除
        _connections.erase(sockfd);
        // 3. 关闭sockfd
        close(sockfd);

        LOG(LogLevel::INFO) << sockfd << " conn del to Reactor";
    }

    void LoopOnce(int timeout)
    {
        int n = _epoller->Wait(_revs, size, timeout);
        for (int i = 0; i < n; i++)
        {
            int sockfd = _revs[i].data.fd;
            uint32_t events = _revs[i].events;

            // 关注到读写事件上-->统一报错
            if (events & EPOLLHUP)
                events = (EPOLLIN | EPOLLOUT);
            if (events & EPOLLERR)
                events = (EPOLLIN | EPOLLOUT);

            // 就绪了
            if ((events & EPOLLIN) && IsExist(sockfd))
            {
                _connections[sockfd]->Recver();
            }
            if ((events & EPOLLOUT) && IsExist(sockfd))
            {
                _connections[sockfd]->Sender();
            }
        }
    }

    void ShowConnection()
    {
        std::cout << "#############################" << std::endl;
        for (auto &conn : _connections)
        {
            std::cout << conn.second->Sockfd() << " : "
                      << Event2String(conn.second->Events()) << std::endl;
        }
        std::cout << "#############################" << std::endl;
    }

    void Dispatcher()
    {
        int timeout = 1000;
        for (;;)
        {
            // 1. 处理事件
            LoopOnce(timeout);
            // 2. 连接管理
            ShowConnection();
    }
    }
    ~Reactor() {}

private:
    std::unordered_map<int, std::shared_ptr<Connection>> _connections; // 连接管理
    std::unique_ptr<Epoller> _epoller;                                 // epoll管理
    struct epoll_event _revs[size];                                    // epoll事件数组
};
```

## Main.cc
```cpp
#include <iostream>
#include <memory>

#include "Calculator.hpp"
#include "Protocol.hpp"
#include "Parser.hpp"
#include "Logger.hpp"
#include "Reactor.hpp"
#include "Listener.hpp"

void Usage(std::string proc)
{
    std::cerr << "Usage: " << proc << " localport" << std::endl;
}

int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        Usage(argv[0]);
        exit(0);
    }

    uint16_t serverport = std::stoi(argv[1]);

    //  1. 业务对象
    std::unique_ptr<Calculator> cal = std::make_unique<Calculator>();

    // 2. 协议和解析协议对象
    std::unique_ptr<Parser> parse_protocol = std::make_unique<Parser>(
        [&cal](Request &req)->Response{
            return cal->Exec(req);
        }
    );

    // 3. 连接管理器 - Listener
    std::shared_ptr<Connection> listener = std::make_shared<Listener>(serverport);  /* 在使用的时候记住需要把fd设置成非阻塞*/
    listener->SetCallBack([&parse_protocol](std::string &inbuffer)->std::string{
        return parse_protocol->Parse(inbuffer);
    });

    // 4. 构建一个Reactor容器
    std::unique_ptr<Reactor> R = std::make_unique<Reactor>();

    // 给reactor中，把连接管理器添加到Reactor
    R->AddConnection(listener);

    // 启动Reactor
    R->Dispatcher();
    
    return 0;
}
```

## Client.cc
```cpp
#include <iostream>
#include <string>
#include <memory>

#include "Socket.hpp"
#include "InetAddr.hpp"
#include "Protocol.hpp"

void Usage(std::string proc)
{
    std::cerr << "Usage: " << proc << " <serverip> <serverport>" << std::endl;
}

int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        Usage(argv[0]);
        exit(0);
    }

    std::string serverip = argv[1];
    uint16_t serverport = std::stoi(argv[2]);

    std::unique_ptr<Socket> sockptr = std::make_unique<TcpSocket>();
    sockptr->BuildClientSocketMethod();

    InetAddr server(serverport, serverip);
    if (sockptr->Connect(server)) // 连接
    {
        std::string inbuffer;
        for (;;)
        {
            // 1. 构建请求
            Request req;
            int x, y;
            char oper;

            std::cout << "Please Enter X:";
            std::cin >> x;
            req.SetX(x);

            std::cout << "Please Enter Y:";
            std::cin >> y;
            req.SetY(y);

            std::cout << "Please Enter Oper:";
            std::cin >> oper;
            req.SetOper(oper);

            // 2. 序列化
            std::string jsonstr;
            req.Serialize(&jsonstr);
            // std::cout << "jsonstr: \r\n" << jsonstr << std::endl;

            // 3 打包
            std::string sendstr = ProtoCol::Package(jsonstr);
            // std::cout << "sendstr: \r\n" << sendstr << std::endl;

            // 4. 发送
            sockptr->Send(sendstr);

            // 5. 接收
            sockptr->Recv(&inbuffer);

            // 6. 反序列化
            std::string package;
            int n = ProtoCol::Unpack(inbuffer, &package);
            if (n > 0)
            {
                Response resp;
                bool r = resp.Deserialize(package);
                if (r)
                {
                    resp.Print();
                }
            }
        }
    }
    return 0;
}
```

## makefile
```cpp
all:reactor_server client

reactor_server:Main.cc
	g++ -o $@ $^ -std=c++17 -ljsoncpp

client:Client.cc
	g++ -o $@ $^ -std=c++17 -ljsoncpp

.PHONY:clean
clean:
	rm -f reactor_server client
```

如果想把这个服务器改成多多进程多线程，其实可以直接创建多进程，每个进程里都搞一个TcpServer，或者说创建多线程，每一个线程里都搞一个Tcpserver，把_listensock套接字添加到其中一个线程的Reactor里，一旦有连接就绪的时候，不是要执行Accepter吗，执行Accepter的时候就不仅仅是 AddConnection了，而是尝试把这个连接添加到哪一个线程的epoll中， 然后就在这个线程里把这个文件描述符处理完。

还可以在Connectoin中设置一个lasttime记录最近访问时间，每一次读或写的时候我们都更新一个对应时间戳，所以只要读或写就绪了就可以更新一下对应Connection最近时间，换句话说此时我们就可以在派发事件后当所有事件处理完了，就可以在unordered_map遍历所有的连接，计算每一个连接已经有多长时间没有动了，因为每一个连接都有自己的最近访问时间，每一次访问都会更新，不更新就是最开始的，所以我们可以获取当前时间在减去Connectoin里保存的历史最近访问时间，计算出时间差，然后就可以所以连接进行连接管理。时间超过5分钟都没有访问过的，服务器就直接把你关掉。

