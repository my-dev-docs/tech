# 6장. 투영에 대해서 2/2

앞 장의 코드 중에서 정점 배열을 설정하는 부분을 살펴 보겠습니다.

```objectivec
-(void)renderView  
{  
    …  

    //: 정점배열을 설정한다  
    glVertexPointer(3, GL_FLOAT, 0, points);  
    glEnableClientState(GL_VERTEX_ARRAY);  
    {  
        //: 점의 크기를 설정한다  
        glPointSize(10);  
        //: 점의 색상을 설정한다  
        glColor4f(1.0, 1.0, 1.0, 1.0);  
        //: 정점 배열의 내용을 점으로 그린다  
        glDrawArrays(GL_POINTS, 0, 4);  
    }  
    glDisableClientState(GL_VERTEX_ARRAY);  
}
```

glVertextPointer 는 OpenGL\|ES의 내부 정점 배열에 사용할 여러 정점을 입력하는 함수입니다. 예전 OpenGL 은 폴리곤을 정의할 때 아래와 같은 glBegin\(\) ~ glEnd\(\) 쌍을 이용해 정의할 수 있었습니다. 아래는 삼각형을 정의하는 코드입니다.

```objectivec
glBegin(GL_TRIANGLES);  
    glVertex3f(-1.0f, -0.5f, -4.0f);     
    glVertex3f( 1.0f, -0.5f, -4.0f);     
    glVertex3f( 0.0f,  0.5f, -4.0f);     
glEnd();
```

위처럼 폴리곤 또는 오브젝트를 정의하면 상태머신으로 작동하는 OpenGL 의 내부 구현은 매우 복잡해 집니다. 모바일 장치에서 빠르고 효율 있게 동작해야 하는 OpenGL\|ES 특성상 구현을 복잡하게 하는 glBegin\(\) ~ glEnd\(\) 형식의 오브젝트 정의 방법을 과감하게 없애고 대신 정점 배열 및 정점 버퍼를 사용하는 방법으로 오브젝트를 정의하게 했습니다.

glEnableClientState, glDisableClientState, glVertextPointer, glDrawArrays 함수에 대해 알아 보겠습니다.

> **glEnableClientState, glDisableClientState**
>
> OpenGL 함수 이름을 보면 Client 가 포함된 것이 있습니다. 이는 OpenGL이 서버와 클라이언트 모델로 설계되고 구현되었기 때문입니다. 간략하게 설명하자면 서버에서 이미지를 생성하고 클라이언트에서는 생성된 이미지를 출력하는 방식입니다. 그와 관련해 함수명이 붙여진 것입니다. 더 자세한 내용은 OpenGL 도서 및 가이드를 참고하세요. 위의 두 함수는 클라이언트 측 기능을 사용가능 또는 사용하지 않음으로 설정하는 함수입니다. OpenGL\|ES 는 기본으로 클라이언트 측 기능을 모두 꺼 놓기 때문에 필요한 기능은 glEnableClientState 로 켜 놓고 사용해야 합니다. 어떤 기능이 필요할 때 Enable로 켜고 기능을 사용한 다음에 Disable로 끄고 하는 패턴으로 사용합니다. 이 두 함수로 설정할 수 있는 클라이언트 측 기능은 아래와 같습니다.
>
> **GL\_COLOR\_ARRAY**
>
> 이 기능이 활성화되면 glDrawArrays 및 glDrawElements로 폴리곤을 렌더링할 때 glColorPointer로 설정한 색상 배열을 참고하여 렌더링합니다.
>
> **GL\_NORMAL\_ARRAY**
>
> 이 기능이 활성화되면 glDrawArray 및 glDrawElements로 폴리곤을 렌더링할 때 glNormalPointer로 설정한 법선 배열을 참고하여 렌더링합니다.
>
> **GL\_POINT\_SIZE\_ARRAY\_OES**
>
> 이 기능이 활성화되면 점과 점 스프라이트를 렌더링할 때 점 크기 배열을 참고하여 렌더링합니다.
>
> **GL\_TEXTURE\_COORD\_ARRAY**
>
> 이 기능이 활성화되면 glDrawArray 및 glDrawElements로 폴리곤을 렌더링할 때 glTexCoordPointer로 설정한 텍스춰 좌표 배열을 참고하여 렌더링합니다.
>
> **GL\_VERTEX\_ARRAY**
>
> 이 기능이 활성화되면 glDrawArray 및 glDrawElements로 폰리곤을 렌더링할 때 glVertextPointer로 설정한 정점 배열을 참고하여 렌더링합니다.
>
> **glVertextPointer\(GLint size, GLenum type, GLsizei stride, const GLvoid\* pointer\);**
>
> glVertexPointer 는 정점 좌표 데이터를 담고 있는 배열의 메모리 위치를 설정할 때 사용합니다. 정점 당 좌표의 개수, 좌표의 데이터 종류\(type\) 그리고 한 정점에서 다음 정점까지의 간격\(메모리 간격\) 을 설정하여 정점 데이터를 담고 있는 배열의 특성을 설정합니다.
>
> * size : 정점 당 좌표의 개수를 지정합니다. 2, 3, 4 중 하나가 되어야 합니다. \(x, y, z\) 로 정점이 구성되면 3이 됩니다.
> * type : 정점을 구성하는 좌표의 데이터 타입을 설정합니다. GL\_BYTE, GL\_SHORT, GL\_FIXED 그리고 GL\_FLOAT으로 설정할 수 있습니다.
> * stride : 한 정점에서 다음 정점까지의 간격입니다. 만약 이 값이 0이면 하나의 특성만으로 구성된 정점 배열로 인식합니다.\( 점 그리기 예제는 위치만 있으므로 0 으로 설정했음\)
> * pointer : 정점 데이터를 담고 있는 배열에서 첫 정점의 첫 좌표가 있는 메모리 주소입니다.
>
> **glDrawArrays\(GLenum mode, GLint first, GLsizei count\)**
>
> glDrawArrays는 여러 원시 기하 도형을 렌더링할 때 사용합니다.
>
> * mode : 렌더링할 원시 기하 도형을 설정합니다. GL\_POINTS, GL\_LINE\_STRIP, GL\_LINE\_LOOP, GL\_LINES, GL\_TRIANGLE\_STRIP, GL\_TRIANGLE\_FAN, GL\_TRIANGLES 를 설정할 수 있습니다.
> * fist : 배열의 시작 인덱스를 설정합니다.
> * count : 렌더링할 개수를 설정합니다.

위의 내용에서 stride와 pointer 내용을 예제를 통해 좀 더 살펴보겠습니다. stride 는 다음 정점까지의 간격입니다. 아래와 같이 위치 말고도 법선, 색상, 텍스춰 좌표 등의 속서을 담은 정점을 정의한 후 각 속성들을 각 배열에 설정해 봅시다.

```objectivec
struct vertex  
{  
    float poisiton[3];    // 위치  
    float normal[3];    // 법선  
    float color[3];        // 색상  
    float texture[2];    // 텍슽춰좌표  
};  

vertex vlist[150];  

glVertexPointer(3, GL_FLOAT, sizeof(vertex), &vlist[0] + 0);  
glNormalPointer(3, GL_FLOAT, sizeof(vertex), &vlist[0] + sizeof(float)*3);  
glColorPointer(3, GL_FLOAT, sizeof(vertex), &vlist[0] + sizeof(float)*6);  
glTexturePointer(2, GL_FLOAT, sizeof(vertex), &vlist[0] + sizeof(float)*9);
```

참고로 OpenGL\|ES 1.x 함수 설명은 [크로노스 공식 문서](http://www.khronos.org/opengles/sdk/1.1/docs/man/)에서 찾아 볼 수 있습니다.

