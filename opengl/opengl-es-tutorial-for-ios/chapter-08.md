# 8장. 색상칠하기

이전 장에서 그렸던 폴리곤에 색을 칠해보려고 합니다. 색을 칠하기 위해서 해야할 일은 많지 않습니다. 정점 배열을 설정하 듯 색상 배열을 설정하면 폴리곤에 색이 표현됩니다. 색을 표현할 때 칠하는 방법이 두 가지가 있는데요 단색으로 칠할 것인가 아니면 색상을 보간해서 혼합해서 칠할 것인가 두 가지 방법이 있습니다.

* GL\_FLAT
  * 색상을 단색으로 칠하며 마지막에 지정한 색상으로 폴리곤을 칠합니다.
* GL\_SMOOTH
  * 각 정점에 정한 색상을 보간하여 혼합하여 칠합니다.

위의 색상 칠하는 방법은 glShadeMode\(GLenum mode\) 함수로 설정해 주며 기본으로 GL\_SMOOTH로 설정되어 있습니다. 그럼 이제 코드를 작성해 보겠습니다. 삼각형을 하나 만들고 각 정점마다 R,G,B 값을 주어 GL\_SMOOTH로 색을 칠해 보겠습니다. 우선 정점을 선언합니다.

```text
GLfloat pointsForGL_TRIANGLE_STRIP[] = {  
    0.2, 0.8, 0.0,  //v1  
    0.2, 0.2, 0.0,  //v2  
    0.8, 0.8, 0.0,  //v3  
};
```

그리고 각 정점에 정할 색상 배열을 선언합니다.

```text
GLfloat colorsForGL_TRIANGLE_STRIP[] = {  
    1.0, 0.0, 0.0, 1.0, //R  
    0.0, 1.0, 0.0, 1.0, //G  
    0.0, 0.0, 1.0, 1.0, //B  
};
```

이제 renderView를 아래와 같이 작성합니다.

```text
-(void)renderView  
{  
    //: 배경을 검은색으로 지운다  
    glClearColor(0.0, 0.0, 0.0, 1.0);  
    glClear(GL_COLOR_BUFFER_BIT);  

    //: 행렬 모드는 모델뷰 행렬로 변경한다  
    glMatrixMode(GL_MODELVIEW);  
    //: 모델뷰 행렬을 초기화한다  
    glLoadIdentity();  

    //: 정점배열을 설정한다  
    glVertexPointer(3, GL_FLOAT, 0, pointsForGL_TRIANGLE_STRIP);  
    //: 색상배열을 설정한다  
    glColorPointer(4, GL_FLOAT, 0, colorsForGL_TRIANGLE_STRIP);  

    //: 색상 칠하기 방법을 설정한다  
    glShadeModel(GL_SMOOTH);  

    //: 색상 배열 사용을 ON  
    glEnableClientState(GL_COLOR_ARRAY);  
    //: 정점 배열 사용을 ON  
    glEnableClientState(GL_VERTEX_ARRAY);  
    {  
        //: 삼각형 3개를 그린다. 처리할 정점의 개수는 3개  
        glDrawArrays(GL_TRIANGLE_STRIP, 0, 3);  
    }  
    //: 정점 배열 사용을 OFF  
    glDisableClientState(GL_VERTEX_ARRAY);  
    //: 색상 배열 사용을 OFF  
    glDisableClientState(GL_COLOR_ARRAY);  
}
```

정점 배열을 설정하는 것과 같이 glColorPointer 함수로 색상 배열을 설정해 줍니다. 그리고 색상 배열을 사용하겠다고 glEnableClientState\(\) 함수로 OpenGL상태머신을 변경합니다 그리고 삼각형을 그려줍니다. 그러면 아래와 같이 예쁘게 색칠된 삼각형을 볼 수 있습니다.

![](../../.gitbook/assets/tut01%20%281%29.png)

이제 glShadeMode에 GL\_FLAT을 주어 빌드 후 실행하면 아래와 같이 마지막에 지정한 파란색으로 삼각형이 칠해지는 것을 확인할 수 있습니다.

```objectivec
-(void)renderView  
{  
    //: 배경을 검은색으로 지운다  
    glClearColor(0.0, 0.0, 0.0, 1.0);  
    glClear(GL_COLOR_BUFFER_BIT);  

    //: 행렬 모드는 모델뷰 행렬로 변경한다  
    glMatrixMode(GL_MODELVIEW);  
    //: 모델뷰 행렬을 초기화한다  
    glLoadIdentity();  

    //: 정점배열을 설정한다  
    glVertexPointer(3, GL_FLOAT, 0, pointsForGL_TRIANGLE_STRIP);  
    //: 색상배열을 설정한다  
    glColorPointer(4, GL_FLOAT, 0, colorsForGL_TRIANGLE_STRIP);  

    //: 색상 칠하기 방법을 설정한다  
    glShadeModel(GL_FLAT);  

    //: 색상 배열 사용을 ON  
    glEnableClientState(GL_COLOR_ARRAY);  
    //: 정점 배열 사용을 ON  
    glEnableClientState(GL_VERTEX_ARRAY);  
    {  
        //: 삼각형 3개를 그린다. 처리할 정점의 개수는 3개  
        glDrawArrays(GL_TRIANGLE_STRIP, 0, 3);  
    }  
    //: 정점 배열 사용을 OFF  
    glDisableClientState(GL_VERTEX_ARRAY);  
    //: 색상 배열 사용을 OFF  
    glDisableClientState(GL_COLOR_ARRAY);  
}
```

![](../../.gitbook/assets/tut02%20%281%29.png)

지금까지 정점배열과 색상배열을 따로 선언해서 사용했는데 만약 같이 사용하려면 어떻게 해야할까요? 아래와 같이 정점 배열을 선언합니다.

```text
GLfloat verticesForGL_TRIANGLE_STRIP[] = {  
    0.2, 0.8, 0.0,          //v1  
    1.0, 0.0, 0.0, 1.0,     //R  

    0.2, 0.2, 0.0,          //v2  
    0.0, 1.0, 0.0, 1.0,     //G  

    0.8, 0.8, 0.0,          //v3  
    0.0, 0.0, 1.0, 1.0,     //B  
};
```

그리고 아래와 같이 renderView를 수정합니다.. glVertexPointer 와 glColorPointer함수만 주의 깊게 보면 됩니다.

```text
-(void)renderView  
{  
    //: 배경을 검은색으로 지운다  
    glClearColor(0.0, 0.0, 0.0, 1.0);  
    glClear(GL_COLOR_BUFFER_BIT);  

    //: 행렬 모드는 모델뷰 행렬로 변경한다  
    glMatrixMode(GL_MODELVIEW);  
    //: 모델뷰 행렬을 초기화한다  
    glLoadIdentity();  

    //: 정점배열을 설정한다  
    glVertexPointer(3, GL_FLOAT, sizeof(GLfloat)*7,  
                    verticesForGL_TRIANGLE_STRIP);  
    //: 색상배열을 설정한다  
    glColorPointer(4, GL_FLOAT, sizeof(GLfloat)*7,  
                    &verticesForGL_TRIANGLE_STRIP[0]+3);  

    //: 색상 칠하기 방법을 설정한다  
    glShadeModel(GL_SMOOTH);  

    //: 색상 배열 사용을 ON  
    glEnableClientState(GL_COLOR_ARRAY);  
    //: 정점 배열 사용을 ON  
    glEnableClientState(GL_VERTEX_ARRAY);  
    {  
        //: 삼각형 3개를 그린다. 처리할 정점의 개수는 3개  
        glDrawArrays(GL_TRIANGLE_STRIP, 0, 3);  
    }  
    //: 정점 배열 사용을 OFF  
    glDisableClientState(GL_VERTEX_ARRAY);  
    //: 색상 배열 사용을 OFF  
    glDisableClientState(GL_COLOR_ARRAY);  
}
```

위의 정점배열에서 정점은 총 3개이고 하나의 정점은 {위치, 색상}으로 구성되어 총 7개의 GLfloat 을 사용합니다. 따라서 stride 는 sizeof\(GLfloat\)\*7 이 되고 정점에서 위치는 정점배열의 시작 주소에 위치 하므로 아래와 같이 코드를 작성하면 됩니다.

```text
//: 정점배열을 설정한다  
glVertexPointer(3, GL_FLOAT, sizeof(GLfloat)*7,  
                    verticesForGL_TRIANGLE_STRIP);
```

그리고 색상배열은 3개의 위치 성분을 지난 다음에 나오므로 아래와 같이 코드를 작성합니다.

```text
//: 색상배열을 설정한다  
    glColorPointer(4, GL_FLOAT, sizeof(GLfloat)*7,  
                   &verticesForGL_TRIANGLE_STRIP[0]+3);
```

위의 코드에서 &verticesForGL\_TRIANGLE\_STRIP\[0\]+3 부분이 아리송한 분은 포인터 연산 부분을 살펴보면 됩니다. 이렇게 한 후 컴파일하고 실행하면 아래와 같이 혼합된 색이 칠해진 삼각형을 볼 수 있습니다.

![](../../.gitbook/assets/tut01%20%281%29.png)

