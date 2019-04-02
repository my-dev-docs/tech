---
description: 'https://github.com/skyfe79/OpenGLTutorial'
---

# 2장. OpenGL 윈도우 프레임웍 만들기 1편

OpenGL 을 배워보는 첫 걸음으로 앞으로 사용하게 될 간단한 윈도우 프레임웤을 만들어 보자. 이 프레임웤의 목적은 윈도우 생성 및 OpenGL 의 설정 그리고 메세지 처리 등의 귀찮은 점을 미리 만들어 놓거나 사용하기 쉽게 하는데 있다. 여기에 소개되는 프레임웤은 초기 버전이므로 앞으로 충분히 수정이 가해질 것이고 간단한 OpenGL 엔진을 작성하는데 사용될 수 있도록 수렴해 가는데 목적이 있다. 물론 여러분들이 자신의 입맛에 맞게 뜯어 고치는 일도 가능하다 ;\]

우선은 OpenGL 에 대한 설정은 빼두고 윈도우 생성 과정을 만들어 보자. 우리가 만들 프레임웤이 만들어지면 아래와 같이 사용된다.

```cpp
#include "eglWindow.h"

int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpszCmdParam, int nCmdShow)
{
    eglWindow app;
    app.Create(hInstance);
    return app.Run();
}
```

코드를 살펴보면 메세지 프로시져\(거의 모든 책에서 WndProc 으로 쓰는 콜백 함수 ;\] \)가 빠져있음을 볼 수 있다. 이 것이 돌아갈 수 있을까? MFC 가 사용하는 메세지맵 매크로 시스템을 이용하면 가능하다. 즉 모든 메세지들마다 일일이 switch - case 문을 작성하지도 않고 메세지맵 매크로 시스템처럼 OnMessage\(\) 의 형식으로 만들 수 있다. 하지만 우리는 MFC 를 사용하지는 않을 것이다. 직접 그 메세지맵 매크로 시스템을 만들어 볼 것이다. 물론 지금은 처음 부분이므로 그 단계는 잠시 뒤로 미루어 두고 위의 eglWindow 클래스가 어떻게 구성되어 있는지 살펴보자. 물론 첫 단계이므로 WndProc 가 필요하다 :\]

다음은 eglWindow 클래스의 코드이다. 초기단계이므로 많은 멤버 함수들이 없다. 당분간은 그럴 것이다. 그 것이 핵심 부분만 이해하는데 더욱 도움이 될 것이다.

```cpp
class eglWindow
{
private:     static BOOL RegisterWindow(HINSTANCE hInstance);
private:     static LRESULT CALLBACK WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam); 
protected:
    HWND mHWnd;
    MSG mMsg;
    HINSTANCE mHInstance;
public:
    eglWindow();
    ~eglWindow();
    BOOL Create(HINSTANCE hInstance, std::string WindowTitle="EDin OpenGL Framework", BOOL bFullScreen=FALSE);
    virtual int Run(void);
};
```

eglWindow 의 멤버함수 하나하나씩 살펴보자.

멤버변수로는 인스턴스의 핸들로 HINSTANCE mHInstance 가 있고, 윈도우 핸들인 HWND mHWnd 그리고 메세지를 받아올 때 필요한 MSG mMsg 가 있다. 이 변수들은 모두 protected: 영역 안의 멤버 변수들로서 eglWindow 가 상속에 사용 될 때 자식 클래스에서 이 멤버 변수들을 사용할 수가 있다. 주목해야할 함수로는 두개의 static 멤버함수이다. static 멤버함수의 성격은 어떠한가? 클래스가 생성되기 전에 프로그램 실행시 우선 실행이 되며 이 static 멤버함수의 안에서는 this 포인터를 사용할 수가 없다. 이는 매우 중요한 성격이다. 꼭 알아두어야하며 이렇게 만드는 이유는 메세지맵 매크로 시스템을 만들면서 알게 될 것이다. 그리고 위의 두개의 static 멤버함수는 private 영역의 함수로 인스턴스를 사용해서 외부에서 접근할 수 없다. 그리고 상속에 사용되도 자식클래스에서 위의 두 static 멤버 함수의 코드는 사용할 수가 없다. 왜 그럴까? 그 이유도 뒤에서 알게될 것이다 ;\]

```cpp
eglWindow::eglWindow() : mHWnd(NULL), mHInstance(NULL)
{
}
```

위의 함수는 eglWindow 의 생성자로 멤버변수들을 초기화 해주고 있다. 다음은 Create 함수를 살펴보자.

```cpp
BOOL eglWindow::Create(HINSTANCE hInstance, std::string WindowTitle, BOOL bFullScreen)
{
    if(!eglWindow::RegisterWindow(hInstance))
        return FALSE;

    mHInstance = hInstance;
    mHWnd = CreateWindowEx(
        0,
        "EdineglWindowClass",
        WindowTitle.c_str(),
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT,
        CW_USEDEFAULT, CW_USEDEFAULT,
        (HWND)NULL,
        (HMENU)NULL,
        hInstance,
        NULL);

    if(!mHWnd) 
    {
        return FALSE;
    }

    ShowWindow(mHWnd, SW_SHOW);
    UpdateWindow(mHWnd);
    return TRUE;
}
```

위의 코드를 보면 내용은 API 를 잘 아는 사람들은 알겠지만 단순히 윈도우즈를 생성해 주는 내용이다. 하지만 주목할 점은 아직 설명하지 않은 RegisterWindow\(\) 함수를 호출한다는 것이다. static 멤버 함수의 호출형식은 네임스페이스::함수이름 이 된다. 자세한 것은 C++ 문법책을 보도록하자. 그럼 RegisterWindow 함수는 어떻게 구성되어 있을까?

```cpp
BOOL eglWindow::RegisterWindow(HINSTANCE hInstance)
{
    WNDCLASSEX wnd;
    wnd.cbSize = sizeof(WNDCLASSEX);
    wnd.cbWndExtra = 0;
    wnd.cbClsExtra = 0;
    wnd.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);
    wnd.hCursor = LoadCursor(NULL, IDC_ARROW);
    wnd.hIcon = LoadIcon(NULL, IDI_WINLOGO);
    wnd.hIconSm = LoadIcon(NULL, IDI_WINLOGO);
    wnd.hInstance = hInstance;
    wnd.lpfnWndProc = eglWindow::WndProc; //! 중요
    wnd.lpszClassName = "EdinCGLWindowClass";
    wnd.lpszMenuName = NULL;
    wnd.style = CS_HREDRAW | CS_VREDRAW;

    if(!::RegisterClassEx(&wnd))
        return FALSE;
    return TRUE;
}
```

위 함수의 내용은 우리가 사용할 윈도우클래스를 시스템에 등록하는 것이다. 하지만 왜 이 함수를 static 함수로 만들어야 하는 것일까? 대답은 간단하다. 멤버함수를 호출하려면 어떻게 해야하는가?

잠시만 생각해보자. 멤버함수를 호출하려면 반드시 해당 클래스의 인스턴스를 만들어야만 멤버 함수를 호출할 수 있다. 그리고 명시적으로 호출을 해줘야한다. 즉 우리가 만든 어떤 클래스의 인스턴스가 app 일 때 app.RegisterWindow\(..\) 를 꼭 해줘야하는 것이다.

하지만 우리에게 필요한 RegisterWindow 함수는 호출되건 말건 상관없는 함수가 아니라 반드시 호출되어야할 함수이다. 그렇다면 프로그래머가 호출에 대해 걱정하지 않아도 자동으로 호출 되게끔 할 수는 없을까?

답은 바로 static 멤버 함수를 사용하는 것이다. 그렇다면 인스턴스를 만들어서 명시적으로 호출하지 않아도 된다. 시스템이 프로그램 실행시 자동으로 호출해 준다. 따라서 위의 static RegisterWindow 멤버 함수는 자동으로 호출되어 우리가 사용할 윈도우클래스를 시스템에 등록해준다.

위에서 굵은 글씨\(중요 주석을 달아 놓은\)로 적혀 있는 코드는 WndProc 를 한 클래스의 멤버함수로 지정한다. 하지만 WndProc 도 static 함수이다. 그 이유는 RegisterWindow 함수가 자동으로 실행될 때의 상황은 eglWindow 의 인스턴스가 메모리에 만들어진 상태가 아니기 때문에 static 멤버 함수인 RegisterWindow 에서 일반 WndProc 멤버 함수를 호출할 수 없다.

따라서 이를 해결하기 위해서는 WndProc 도 static 멤버 함수로 만들어 프로그램 실행시 메모리에 코드를 올려 놓도록 한다. 그러면 당연히 RegisterWindow 함수에서 WndProc 함수의 포인터\(코드가 올려져 있는 메모리 주소\)를 얻을 수 있다.

eglWindow 의 static 멤버 함수 WndProc 의 코드는 아래와 같다.

```cpp
LRESULT CALLBACK eglWindow::WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam)
{
    switch(iMessage)
    {
    case WM_LBUTTONDOWN:
        MessageBox(hWnd, TEXT("EDin OpenGL Framework 0.1"), TEXT("HELLO:)"), MB_OK);
        break;
    case WM_SIZE:
        MessageBeep(-1);
        break;
    case WM_DESTROY:
        PostQuitMessage(0);
        break;
    }
    return DefWindowProc(hWnd, iMessage, wParam, lParam);
}
```

Run 의 멤버 함수는 단순히 메세지 펌핑을 하는 코드이다.

```cpp
int eglWindow::Run(void)
{
    while(GetMessage(&mMsg, NULL, 0, 0))
    {
        TranslateMessage(&mMsg);
        DispatchMessage(&mMsg);
    }
    return mMsg.message;
}
```

위의 코드를 보았을 때 어떤 윈도우 메세지의 핸들링을 하려할 때 매번 eglWindow::WndProc 의 코드를 고쳐야 한다. 그리고 코드가 길어지면 길어 질 수록 WndProc 의 코드를 읽기\(분석\)가 힘들어 질 것이다. 이럴 때 MFC 의 메세지 맵 같은 시스템을 사용할 수 있으면 얼마나 편할까? 지금부터 그 메세지맵 매크로 시스템을 만들어 보자. 잠시 눈을 돌려 MFC 의 클래스 코드를 보면 다음과 같은 형태로 메세지 매크로가 쓰이는 것을 볼 수 있다.

```cpp
class CMyDialog : public CDialog
{
    .....
    .....
    DECLARE_MESSAGE_MAP() 
};

BEGIN_MESSAGE_MAP(CMyDialog, CDialog)
    ON_MESSAGE(...)
END_MESSAGE_MAP();
```

위의 DECLARE\_MESSAGE\_MAP, BEGIN\_MESSAGE\_MAP, END\_MESSAGE\_MAP 등은 위에서 우리가 작성했던 코드들을 매크로로 써주는 역할만을 할 뿐이다. DECLARE\_MESSAGE\_MAP 은 메세지 매크로 시스템에 필요한 함수들과 타입들을 선언하는 역할을 하고 BEGIN\_MESSAGE\_MAP, ON\_MESSAGE, END\_MESSAGE\_MAP 은 DECLARE\_MESSAGE\_MAP 가 선언한 함수를 구현하는 역할을 한다. 그럼 각각의 매크로가 어떻게 구성되는지를 살펴보자.

아래의 코드는 DECLARE\_MESSAGE\_MAP 의 코드이다.

```cpp
#define DECLARE_MESSAGE_MAP(CLASSNAME) \
    typedef void (CLASSNAME::*CLASSNAME##FuncPtr)(WPARAM, LPARAM);\
    typedef struct _MESSAGEMAP\
    {\
        UINT iMsg;\
        CLASSNAME##FuncPtr fp;\
    }MESSAGEMAP;\
    private: static BOOL RegisterWindow(HINSTANCE hInstance);\
    private: static LRESULT CALLBACK WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam);\
    private: static MESSAGEMAP MessageMap[];\
    public:
```

코드가 좀 복잡해 보일지도 모르겠다. 매크로함수와 \#\# 연산자 등은 문법책을 참고하길 바란다. 연결연산자를 제거하고 CLASSNAME 을 eglWindow 의 코드로 바꾸어 살펴보자.

```cpp
#define DECLARE_MESSAGE_MAP(eglWindow) 
    typedef void (eglWindow::*eglWindowFuncPtr)(WPARAM, LPARAM);
    typedef struct _MESSAGEMAP
    {
        UINT iMsg;
        eglWindowFuncPtr fp;
    }MESSAGEMAP;
    private: static BOOL RegisterWindow(HINSTANCE hInstance);
    private: static LRESULT CALLBACK WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam);
    private: static MESSAGEMAP MessageMap[];
    public:
```

함수포인터의 타입 정의와 MESSAGEMAP 구조체를 제외하면 위에서 보았던 클래스의 일부분임을 금새 눈치챘을 것이다. typedef void \(eglWindow::\*eglWindowFuncPtr\)\(WPARAM, LPARAM\); 로 된 타입 정의문은 eglWindow 의 멤버함수 중 리턴값은 void 이고 인자는 WPARAM 과 LPARAM 을 받는 함수를 가리키는 포인터 타입을 정의한 것이다. 그리고 MESSAGEMAP 구조체는 메세지와 관련된 함수를 연결해 놓는 자료형으로 BEGIN\_MESSAGE\_MAP 과 ON\_MESSAGE 로 초기화 된다. 이 예는 매크로의 CLASSNAME 이 eglWindow 우 일 때의 경우를 보여주는 것이다. 다른 클래스에서 필요하다면 CLASSNAME 에 그 클래스의 이름을 써 주면 위의 자료형이 선언될 것이다. 여기서 RegisterWindow 와 WndProc 멤버 함수들이 private 영역에 포함된 것은 메세지 처리가 필요한 모든 클래스에서는 각각의 RegisterWindow 와 WndProc 들이 필요하기 때문이다. 상속에서도 이는 마찬가지인데 WndProc 만 코드를 공유할 수 없지만 메세지 핸들링 함수는 코드를 공유할 수가 있다. 물론 공유하지 않을 수도 있다. 뒷부분에서 상속에서 사용하는 예제를 보이겠다.

다음은 BEGIN\_MESSAGE\_MAP 과 ON\_MESSAGE , END\_MESSAGE\_MAP 을 살펴보자.

```cpp
#define BEGIN_MESSAGE_MAP(CLASSNAME) \
    BOOL CLASSNAME::RegisterWindow(HINSTANCE hInstance)\
    {\
        WNDCLASSEX wnd;\
        wnd.cbSize = sizeof(WNDCLASSEX);\
        wnd.cbWndExtra = 0;\
        wnd.cbClsExtra = 0;\
        wnd.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);\
        wnd.hCursor = LoadCursor(NULL, IDC_ARROW);\
        wnd.hIcon = LoadIcon(NULL, IDI_WINLOGO);\
        wnd.hIconSm = LoadIcon(NULL, IDI_WINLOGO);\
        wnd.hInstance = hInstance;\
        wnd.lpfnWndProc = CLASSNAME::WndProc;\
        wnd.lpszClassName = "EdineglWindowClass";\
        wnd.lpszMenuName = NULL;\
        wnd.style = CS_HREDRAW | CS_VREDRAW;\
        if(!::RegisterClassEx(&wnd))\
            return FALSE;\
        return TRUE;\
    }\
    LRESULT CALLBACK CLASSNAME::WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam)\
    {\
        int i=0;\
        static CLASSNAME ido;
       while(MessageMap[i].iMsg != 0)\
        {\
            if(iMessage == MessageMap[i].iMsg)\
            {\
                (ido.*(MessageMap[i].fp))(wParam, lParam);\
            }\
            ++i;\
        }\
        return DefWindowProc(hWnd, iMessage, wParam, lParam);\
    }\
    CLASSNAME::MESSAGEMAP CLASSNAME::MessageMap[] = {
```

아래는 코드의 이해를 돕기 위해서 CLASSNAME 을 eglWindow 로 바꾼 것이다.

```cpp
#define BEGIN_MESSAGE_MAP(eglWindow) 
    BOOL eglWindow::RegisterWindow(HINSTANCE hInstance)
    {
        WNDCLASSEX wnd;
        wnd.cbSize = sizeof(WNDCLASSEX);
        wnd.cbWndExtra = 0;
        wnd.cbClsExtra = 0;
        wnd.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);
        wnd.hCursor = LoadCursor(NULL, IDC_ARROW);
        wnd.hIcon = LoadIcon(NULL, IDI_WINLOGO);
        wnd.hIconSm = LoadIcon(NULL, IDI_WINLOGO);
        wnd.hInstance = hInstance;
        wnd.lpfnWndProc = eglWindow::WndProc;
        wnd.lpszClassName = "EdineglWindowClass";
        wnd.lpszMenuName = NULL;
        wnd.style = CS_HREDRAW | CS_VREDRAW;
        if(!::RegisterClassEx(&wnd))
            return FALSE;
        return TRUE;
    }
    LRESULT CALLBACK eglWindow::WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam)
    {
        int i=0;
        static eglWindow ido;
        while(MessageMap[i].iMsg != 0)
        {
            if(iMessage == MessageMap[i].iMsg)
            {
                (ido.*(MessageMap[i].fp))(wParam, lParam);
            }
            ++i;
        }
        return DefWindowProc(hWnd, iMessage, wParam, lParam);
    }
    eglWindow::MESSAGEMAP eglWindow::MessageMap[] = {
```

위의 내용은 BEGIN\_MESSAGE\_MAP 매크로의 내용이다. 그냥 RegisterWindow 와 WndProc 의 함수 정의만 있을 뿐이며 맨 끝줄에 eglWindow::MESSAGEMAP eglWindow::MessageMap\[\] = { 로 MessageMap\[\] 구조체 배열의 초기화를 하는 부분이 있다. 초기화 부분은 뒤에서 보면 알겠지만 ON\_MESSAGE 가 처리한다. 

그런데 코드를 보면 이상한 부분이 있다. 어디일까? 굵게 쓰인 코드가 보일 것이다. 위의 코드는 무엇이 잘 못되었을까? 그것은 static eglWindow 의 인스턴스 ido 와 우리가 프로그램에서 사용하는 인스턴스\(this\) 는 전혀 다른 객체라는 것이다. 

즉 함수의 코드는 공유할 수 있되 멤버 변수는 공유를 할 수 없다. 메세지 처리 등을 해보면 별 문제 없이 작동하지만 변수조작등을 가하면 반드시 벌레가 생길 것이다. 그렇다면 어떻게 해야하는가? \(ido._\(MessageMap\[i\].fp\)\)\(wParam, lParam\); 에서 ido 를 WndProc 를 호출하는, 즉 this 포인터로 바꾸어 호출해야 한다. \(this-&gt;_\(MessageMap\[i\].fp\)\)\(wParam, lParam\); 으로 바꾸어야한다. \(이 코드는 함수포인터를 실행하는 코드이다. 함수포인터에 대한 내용은 문법책을 참고하길 바란다\) 

하지만 WndProc 는 무슨 함수인가? 바로 static 함수이다! static 함수는 위에서 말했듯이 this 포인터를 사용할 수 없다. 그렇다면 두 눈 시퍼렇게 뜨고 벌레가 생기는 걸 보고 있어야만 할까? 당연히 아니다! 해결법이 있다. this 포인터를 사용할 수 있는 방법이 있다. 

해결법은 WNDCLASSEX 구조체에 숨어 있다. 바로 여분의 데이타를 저장할 수 있도록 한 cbWndExtra 와 cbClsExtra 를 사용하면 된다. 둘 다 사용하는 것이 아니라 여기서는 cbWndExtra 를 사용하기로 하겠다. 아래는 이를 적용한 코드로 굵게 쓰여진 코드를 주의 깊에 살펴 보자.

```cpp
BOOL eglWindow::RegisterWindow(HINSTANCE hInstance)
    {
        WNDCLASSEX wnd;
        wnd.cbSize = sizeof(WNDCLASSEX);
        wnd.cbWndExtra = 4;
        wnd.cbClsExtra = 0;
        wnd.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);
        wnd.hCursor = LoadCursor(NULL, IDC_ARROW);
        wnd.hIcon = LoadIcon(NULL, IDI_WINLOGO);
        wnd.hIconSm = LoadIcon(NULL, IDI_WINLOGO);
        wnd.hInstance = hInstance;
        wnd.lpfnWndProc = eglWindow::WndProc;
        wnd.lpszClassName = "EdineglWindowClass";
        wnd.lpszMenuName = NULL;
        wnd.style = CS_HREDRAW | CS_VREDRAW;
        if(!::RegisterClassEx(&wnd))
            return FALSE;
        return TRUE;
    }
```

```cpp
BOOL eglWindow::Create(HINSTANCE hInstance, std::string WindowTitle, BOOL bFullScreen)
{
    if(!eglWindow::RegisterWindow(hInstance))
        return FALSE;
    mHInstance = hInstance;
    mHWnd = CreateWindowEx(
        0,
        "EdineglWindowClass",
        WindowTitle.c_str(),
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT,
        CW_USEDEFAULT, CW_USEDEFAULT,
        (HWND)NULL,
        (HMENU)NULL,
        hInstance,
        (LPVOID)this); //메세지맵을 위하여 this 포인터를 넘긴다.
        if(!mHWnd) 
        {
            return FALSE;
        }
    ShowWindow(mHWnd, SW_SHOW);
    UpdateWindow(mHWnd);
    return TRUE;
}
```

```cpp
LRESULT CALLBACK eglWindow::WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam)
{
    int i=0;
    static eglWindow *ido = (eglWindow *)GetWindowLong(hWnd, GWL_USERDATA);
    if(ido == NULL && iMessage != WM_CREATE)
    {
        return DefWindowProc(hWnd, iMessage, wParam, lParam);
    }
    if(iMessage == WM_CREATE)
    {
        ido = (eglWindow *)(((LPCREATESTRUCT)lParam)->lpCreateParams);
    }
    while(MessageMap[i].iMsg != 0)
    {
        if(iMessage == MessageMap[i].iMsg)
        {
            (ido->*(MessageMap[i].fp))(wParam, lParam);
        }
        ++i;
    }
    return DefWindowProc(hWnd, iMessage, wParam, lParam);
}
LRESULT CALLBACK eglWindow::WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam)
{
    int i=0;
    static eglWindow *ido = (eglWindow *)GetWindowLong(hWnd, GWL_USERDATA);
    if(ido == NULL && iMessage != WM_CREATE)
    {
        return DefWindowProc(hWnd, iMessage, wParam, lParam);
    }
    if(iMessage == WM_CREATE)
    {
        ido = (eglWindow *)(((LPCREATESTRUCT)lParam)->lpCreateParams);
    }
    while(MessageMap[i].iMsg != 0)
    {
        if(iMessage == MessageMap[i].iMsg)
        {
            (ido->*(MessageMap[i].fp))(wParam, lParam);
        }
        ++i;
    }
    return DefWindowProc(hWnd, iMessage, wParam, lParam);
}
```

위의 코드를 보면 RegisterWindow 함수에서 WNDCLASSEX 의 구조체 멤버 변수 중 cbWndExtra 에 4 의 값을 주는데 이 것은 포인터가 4 바이트이기 때문이다. 그리고 이렇게 할당된 공간에 데이타를 넣는 것은 CreateWindowEx 함수에서 하는데 맨 끝에 \(LPVOID\)this 를 넘겨 줌으로써 this 포인터를 저장하게 된다. 이렇게 저장된 값은 윈도우 프로시져 WndProc 에서 WM\_CREATE 메세지가 전달 될 때 lParam 으로 넘어온다. 정확하게 말하자면 lParam 은 CREATESTRUCT 의 포인터를 가지고 있고 CREATESTRUCT 구조체의 멤버 변수중 lpCreateParams 이 여분의 데이타 즉 위에서 저장한 this 포인터의 값을 가지고 있게 되는 것이다. 메세지가 WM\_CREATE 일 때 ido 에 this 포인터의 값을 할당하는 것을 볼 수 있다. 이렇게 함으로써 벌레를 잡을 수 있게 되었다. 그럼 나머지 매크로들을 보자.

```cpp
#define ON_MESSAGE(CLASSNAME, MESSAGE, HANDLER) {MESSAGE, CLASSNAME::HANDLER},
#define END_MESSAGE_MAP() {0, NULL} };
```

위는 ON\_MESSAGE 와 END\_MESSAGE\_MAP 의 매크로 내용이다. BEGIN\_MESSAGE\_MAP 과 ON\_MESSAGE, END\_MESSAGE\_MAP 의 매크로를 합쳐서 쓰면 다음과 같은 MessageMap\[\] 구조체 배열을 초기화하는 코드가 된다.

```cpp
BEGIN_MESSAGE_MAP(eglWindow)
    ON_MESSAGE(eglWindow, WM_LBUTTONDOWN, OnLButtonDown)
    ON_MESSAGE(eglWindow, WM_DESTROY, OnDestroy)
END_MESSAGE_MAP();
```

위의 내용을 코드로 써보면 아래와 같다.

```cpp
eglWindow::MESSAGEMAP eglWindow::MessageMap[] = {
    {WM_LBUTTONDOWN, eglWindow::OnLButtonDown},
    {0, NULL}
};
```

그럼 벌레를 잡은 모든 매크로를 보도록 하자.

```cpp
#ifndef _EGL_MSGMACRO_SYSTEM_H_
#define _EGL_MSGMACRO_SYSTEM_H_

//*************************************************************************************************************//
// Message Macro System 

#define BEGIN_MESSAGE_MAP(CLASSNAME) \
    BOOL CLASSNAME::RegisterWindow(HINSTANCE hInstance)\
    {\
        WNDCLASSEX wnd;\
        wnd.cbSize = sizeof(WNDCLASSEX);\
        wnd.cbWndExtra = 4;\
        wnd.cbClsExtra = 0;\
        wnd.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);\
        wnd.hCursor = LoadCursor(NULL, IDC_ARROW);\
        wnd.hIcon = LoadIcon(NULL, IDI_WINLOGO);\
        wnd.hIconSm = LoadIcon(NULL, IDI_WINLOGO);\
        wnd.hInstance = hInstance;\
        wnd.lpfnWndProc = CLASSNAME::WndProc;\
        wnd.lpszClassName = "EdineglWindowClass";\
        wnd.lpszMenuName = NULL;\
        wnd.style = CS_HREDRAW | CS_VREDRAW;\
        if(!::RegisterClassEx(&wnd))\
            return FALSE;\
        return TRUE;\
    }\
    LRESULT CALLBACK CLASSNAME::WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam)\
    {\
        int i=0;\
        static CLASSNAME *ido = (CLASSNAME *)GetWindowLong(hWnd, GWL_USERDATA);\
        if(ido == NULL && iMessage != WM_CREATE)\
        {\
            return DefWindowProc(hWnd, iMessage, wParam, lParam);\
        }\
        if(iMessage == WM_CREATE)\
        {\
            ido = (CLASSNAME *)(((LPCREATESTRUCT)lParam)->lpCreateParams);\
        }\
        while(MessageMap[i].iMsg != 0)\
        {\
            if(iMessage == MessageMap[i].iMsg)\
            {\
                (ido->*(MessageMap[i].fp))(wParam, lParam);\
            }\
            ++i;\
        }\
        return DefWindowProc(hWnd, iMessage, wParam, lParam);\
    }\
    CLASSNAME::MESSAGEMAP CLASSNAME::MessageMap[] = {
#define ON_MESSAGE(CLASSNAME, MESSAGE, HANDLER) {MESSAGE, CLASSNAME::HANDLER},
#define END_MESSAGE_MAP() {0, NULL} };

#define DECLARE_MESSAGE_MAP(CLASSNAME) \
    typedef void (CLASSNAME::*CLASSNAME##FuncPtr)(WPARAM, LPARAM);\
    typedef struct _MESSAGEMAP\
    {\
        UINT iMsg;\
        CLASSNAME##FuncPtr fp;\
    }MESSAGEMAP;\
    private: static BOOL RegisterWindow(HINSTANCE hInstance);\
    private: static LRESULT CALLBACK WndProc(HWND hWnd, UINT iMessage, WPARAM wParam, LPARAM lParam);\
    private: static MESSAGEMAP MessageMap[];\
    public: 

// Message Macro System
//****************************************************************************************************************//

#endif
```

이를 이용해 맨처음 만들었던 예제를 다시 만들어 보면 아래와 같다.

```cpp
// 클래스 선언부분
class eglWindow
{
protected:
    HWND mHWnd;
    MSG mMsg;
    HDC mDC;
    HINSTANCE mHInstance;
public:
    eglWindow();
    ~eglWindow();
    BOOL Create(HINSTANCE hInstance, std::string WindowTitle="EDin OpenGL Framework", BOOL bFullScreen=FALSE);
    virtual int Run(void);

    DECLARE_MESSAGE_MAP(eglWindow);

    void OnLButtonDown(WPARAM w, LPARAM l);
    void OnDestroy(WPARAM w, LPARAM l);
};
```

```cpp
//클래스 구현 부분
BEGIN_MESSAGE_MAP(eglWindow)
    ON_MESSAGE(eglWindow, WM_LBUTTONDOWN, OnLButtonDown)
    ON_MESSAGE(eglWindow, WM_DESTROY, OnDestroy)
END_MESSAGE_MAP();

void eglWindow::OnLButtonDown(WPARAM w, LPARAM l)
{
    MessageBox(this->mHWnd, TEXT("EDin's OpenGL Framework 입니다"), TEXT("Hello;)"), MB_OK);
}

void eglWindow::OnDestroy(WPARAM w, LPARAM l)
{
    PostQuitMessage(0);
}

eglWindow::eglWindow() : mDC(NULL), mHWnd(NULL), mHInstance(NULL)
{
}

eglWindow::~eglWindow()
{
}

BOOL eglWindow::Create(HINSTANCE hInstance, std::string WindowTitle, BOOL bFullScreen)
{
    if(!eglWindow::RegisterWindow(hInstance))
        return FALSE;

    mHInstance = hInstance;
    mHWnd = CreateWindowEx(
        0,
        "EdineglWindowClass",
        WindowTitle.c_str(),
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT,
        CW_USEDEFAULT, CW_USEDEFAULT,
        (HWND)NULL,
        (HMENU)NULL,
        hInstance,
        (LPVOID)this); //메세지맵을 위하여 this 포인터를 넘긴다.

    if(!mHWnd) 
    {
        return FALSE;
    }

    ShowWindow(mHWnd, SW_SHOW);
    UpdateWindow(mHWnd);
    return TRUE;
}

int eglWindow::Run(void)
{
    while(GetMessage(&mMsg, NULL, 0, 0))
    {
        TranslateMessage(&mMsg);
        DispatchMessage(&mMsg);
    }
    return mMsg.message;
}
```

```cpp
//메인프로그램
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd)
{
    eglWindow app;
    app.Create(hInstance);
    return app.Run();
}
```

위의 코드를 보면 WndProc 부분이 사라졌음을 볼 수 있다. 그런데 매 클래스 작성마다 Create 함수와 Run 함수등을 만들어야할까? 그렇다면 이런 프레임워크를 만들지도 않았을 것이다. 즉 eglWindow 에 필요한 것은 다 만들어 놓고 재정의 필요할지도 모르는 함수에는 가상함수를 사용하면 그만이다. 위에서 나중에 설명한다고 했던 상속에서 메세지 핸들러에 대한 코드 공유를 살펴 보자.

위에서는 마우스 왼쪽 버튼을 눌렀을 때 메세지 박스가 뜨는데 이 것을 비프음\(띵~\)이 들리게 할려면 어떻게 할까? 또는 상속함수에서도 그대로 메세지 박스가 보이게 하려면 어떻게 할까? 첫 질문에 대한 답은 바로 가상함수를 사용하는 것이다.

```cpp
//클래스 선언 부분
class eglWindow
{
protected:
    HWND mHWnd;
    MSG mMsg;
    HDC mDC;
    HINSTANCE mHInstance;
public:
    eglWindow();
    ~eglWindow();
    BOOL Create(HINSTANCE hInstance, std::string WindowTitle="EDin OpenGL Framework", BOOL bFullScreen=FALSE);
    virtual int Run(void);

    DECLARE_MESSAGE_MAP(eglWindow);

    virtual void OnLButtonDown(WPARAM w, LPARAM l);
    virtual void OnDestroy(WPARAM w, LPARAM l);
};

//클래스 구현 부분
BEGIN_MESSAGE_MAP(eglWindow)
    ON_MESSAGE(eglWindow, WM_LBUTTONDOWN, OnLButtonDown)
    ON_MESSAGE(eglWindow, WM_DESTROY, OnDestroy)
END_MESSAGE_MAP();

void eglWindow::OnLButtonDown(WPARAM w, LPARAM l)
{
    MessageBox(this->mHWnd, TEXT("EDin's OpenGL Framework 입니다"), TEXT("Hello;)"), MB_OK);
}

void eglWindow::OnDestroy(WPARAM w, LPARAM l)
{
    PostQuitMessage(0);
}
```

```cpp
//클래스 선언 부분
class eglSubWindow : public eglWindow
{
    DECLARE_MESSAGE_MAP(eglSubWindow);
public:
    virtual void OnLButtonDown(WPARAM w, LPARAM l);
};

BEGIN_MESSAGE_MAP(eglSubWindow)
    ON_MESSAGE(eglSubWindow, WM_LBUTTONDOWN, OnLButtonDown)
    ON_MESSAGE(eglSubWindow, WM_DESTROY, OnDestroy)
END_MESSAGE_MAP();

void eglSubWindow::OnLButtonDown(WPARAM w, LPARAM l)
{
    MessageBeep(-1);
}
```

위의 코드를 보면 eglWindow 의 메세지 핸들러의 함수가 virtual 즉 가상함수로 선언되어 있다. 그리고 OnLButtonDown 과 OnDestroy 의 메세지 핸들러를 정의하고 있다. eglWindow 를 상속받는 eglSubWindow 는 OnLButtonDown 만 정의하고 있다. 내용은 마우스 왼쪽 버튼을 눌렀을 때 비프음이 들리는 것이다. 하지만 OnDestroy 의 핸들러는 없는데 메세지맵에는 ON\_MESSAGE\(eglSubWindow, WM\_DESTROY, OnDestroy\) 으로 핸들러를 등록하고 있다. 이는 상속에서 eglWindow 의 public 함수인 OnDestroy 가 eglSubWindow 에 상속되기 때문이며\(코드공유\) 마우스왼쪽 버튼을 눌렀을 때 우리가 원하는 기능을 수행하는 함수를 다시 만들 수 있도록\(코드재정의\) 가상함수를 사용했다.

```cpp
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd)
{
    eglSubWindow app;
    app.Create(hInstance);
    return app.Run();
}
```

위의 코드를 실행하면 이번에는 마우스 왼쪽 버튼을 누르면 메세지 박스가 나타나는 대신에 비프음이 들릴 것이고 프로그램 종료는 OnDestroy 의 코드 공유로 전 단계의 프로그램과 똑같이 작동한다.

이상으로 OpenGL 을 제외하고 윈도우 생성과 메세지 핸들링에 관한 프레임웤을 만들어 보았다. 너무나 미약한 내용이지만 이 글이 많은 사람들에게 도움이 되었으면 좋겠다. 이 글의 다음 내용에서는 OpenGL 관련 책 중 어느서나 다루는 OpenGL 윈도우 생성에 대한 코드가 나올 것이다. 이를 클래스화 해 본다.

