# 9장. 텍스춰맵핑

폴리곤에 이미지를 맵핑하는 텍스춰맵핑을 다루려고 합니다. 텍스춰맵핑이 가능해야 디자이너가 그려준 멋진 그림을 화면에 그릴 수 있습니다. 텍스춰맵핑은 아래의 순서대로 이뤄집니다.

1. 텍스춰로 사용할 이미지를 불러오고 이미지로 부터 데이터 바이트 배열을 얻어온다.
2. 텍스춰를 생성하고 텍스춰이름\(텍스춰구분자\)를 얻는다.
3. 텍스춰를 바인딩하고 텍스춰 좌표를 활성화한다.
4. 폴리곤을 그린다.

그다지 복잡하지 않습니다. iOS에서 이미지 데이터의 바이트 배열을 얻어 오는 것은 코어그래픽스의 도움을 받아야 합니다. UIImage 에서 바로 해당 이미지의 데이터에 접근할 수 없기 때문입니다. UIImage로 불러온 이미지를 코어그래픽스를 통해 이미지 데이터 배열을 받아오는 순서는 아래와 같습니다.

1. UIImage로 이미지를 불러온다
2. UIImage 에서 CGImageRef 를 얻는다
3. 이미지 데이터를 담을 버퍼와 함께 CGBitmapContext를 만든다
4. CGImage를 CGBitmapContext에 그린다.
5. 이미지 데이터를 담을 버퍼에서 이미지 데이터를 얻을 수 있다.

우선 텍스춰를 맵핑하는 코드를 작성하기 전에 UIImage를 불러와 텍스춰를 생성하는 클래스를 작성해 보겠습니다. XCode에서 새 로운 클래스 생성을 선택하신 다음 NSObject를 상속받는 OGLTexture 클래스를 생성합니다.

```objectivec
#import <Foundation/Foundation.h>  

@interface OGLTexture : NSObject  
{  
    //: 텍스춰 아이디  
    GLuint  textureID;  
    //: 이미지 데이터  
    GLubyte *imageData;  
    //: 이미지 너비, 높이  
    size_t  width;  
    size_t  height;  
}  
@property(nonatomic, readonly) GLuint   textureID;  
@property(nonatomic, readonly) GLubyte  *imageData;  
@property(nonatomic, readonly) size_t   width;  
@property(nonatomic, readonly) size_t   height;  
+(id)textureWithImagePath:(NSString *)path;  
-(id)initWithImagePath:(NSString *)path;  
-(BOOL)bindTexture;  
@end
```

그리고 위와 같이 인터페이스를 작성합니다. 이 인터페이스의 구현부는 아래와 같습니다.

```objectivec
#import "OGLTexture.h"  

@interface OGLTexture()  
-(BOOL)loadUIImage:(NSString *)path;  
-(BOOL)generateTexture;  
-(void)deleteTextre;  
@end  

@implementation OGLTexture  
@synthesize textureID;  
@synthesize imageData;  
@synthesize width;  
@synthesize height;  

+(id)textureWithImagePath:(NSString *)path  
{  
    return [[[self alloc] initWithImagePath:path] autorelease];  
}  

-(id)initWithImagePath:(NSString *)path  
{  
    self = [super init];  
    if(self!=nil)  
    {  
        textureID = -1;  
        imageData = NULL;  
        width     = -1;  
        height    = -1;  
        if([self loadUIImage:path])  
        {  
            [self generateTexture];  
        }  
    }  
    return self;  
}  

-(void)dealloc  
{  
    [self deleteTextre];  
    if(imageData)  
    {  
        free(imageData);  
    }  
    [super dealloc];  
}  

-(BOOL)loadUIImage:(NSString *)path  
{  
    UIImage *image = [UIImage imageWithContentsOfFile:path];  
    if(image == nil)  
        return NO;  

    //: 이미지의 정보를 얻어 온다  
    CGImageRef cgImage = [image CGImage];  
    width = CGImageGetWidth(cgImage);  
    height= CGImageGetHeight(cgImage);  

    //: 이미지의 너비와 높이가 2의 승수인지 살펴봐야 하지만  
    //: 생략한다  

    //: 이미지 데이터의 바이트 배열을 만들기 위해서  
    //: 비트맵 컨텍스트를 만들고 거기에 이미지를 그린다  
    //: 텍스춰로 사용하는 이미지는 모두 RGBA 4바이트로 가정한다  
    if(imageData)  
        free(imageData);  

    imageData = (GLubyte *)calloc(width*height*4, sizeof(GLubyte));  
    if(imageData == NULL)  
        return NO;  

    //: 이미지를 그릴 비트맵컨텍스트를 생성한다  
    CGContextRef bitmapContext = CGBitmapContextCreate(imageData,  
                                                       width,  
                                                       height,  
                                                       8,  
                                                       width*4,  
                                                       CGImageGetColorSpace(cgImage),  
                                                       kCGImageAlphaPremultipliedLast);  

    //: 이미지를 비트맵컨텍스트에 그린다  
    CGContextDrawImage(bitmapContext,  
                       CGRectMake(0, 0, width, height),  
                       cgImage);  

    //: 비트맵데이터만 필요하므로 컨텍스트는 메모리 해제한다  
    CGContextRelease(bitmapContext);  
    return YES;  
}  

/** 
 * @brief 텍스춰를 생성한다 
 */  
-(BOOL)generateTexture  
{  
    if(imageData == NULL)  
        return NO;  

    if(textureID != -1)  
    {  
        glDeleteTextures(1, &textureID);  
        textureID = -1;  
    }  

    //: 텍스춰 구분자를 생성한다  
    glGenTextures(1, &textureID);  

    //: 텍스춰를 바인딩한다  
    glBindTexture(GL_TEXTURE_2D, textureID);  

    //: 텍스춰 파라미터를 설정한다  
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);  
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);  

    //: 텍스춰 구분자와 이미지 데이터를 연결한다  
    glTexImage2D(GL_TEXTURE_2D,  
                 0,  
                 GL_RGBA,  
                 width,  
                 height,  
                 0,  
                 GL_RGBA,  
                 GL_UNSIGNED_BYTE,  
                 imageData);  
    return YES;  
}  
/** 
 * @brief 텍스춰를 삭제한다 
 */  
-(void)deleteTextre  
{  
    if(textureID != -1)  
    {  
        glDeleteTextures(1, &textureID);  
        textureID = -1;  
    }  
}  

/** 
 * @brief 텍스춰를 바인딩한다 
 */  
-(BOOL)bindTexture  
{  
    if(textureID == -1)  
        return NO;  

    //: 텍스춰를 바인딩한다  
    glBindTexture(GL_TEXTURE_2D, textureID);  
    return YES;  
}  
@end
```

위의 코드에서 loadUIImage가 UIImage에서 불러온 이미지를 코어 그래픽스를 통해 이미지 데이터의 바이트 배열을 얻는 코드입니다. 코드의 내용은 주석을 참고하시길 바랍니다. 코어그래픽스의 내용은 훗날 쿼츠 튜토리얼을 작성할 때 다뤄 보도록 하겠습니다.

그 다음 generateTexture 는 텍스춰 구분자를 생성하고 텍스춰 구분자와 이미지 데이터를 연결해 주는 메서드입니다. 이 메서드에서는 아래의 함수들을 사용합니다.

* glGenTextures\(int n, GLuint \*textures\) 이 함수는 n개의 텍스춰 구분자를 생성합니다. 그리고 생성된 텍스춰 구분자는 textures에 담기게 됩니다.
* glBindTexture\(GLenum target, GLuint texture\) 이 함수는 target에 texture를 바인딩합니다. OpenGL\|ES는 타겟으로 GL\_TEXTURE\_2D 만 지원하므로 반드시 GL\_TEXTURE\_2D로 설정해야 합니다. texture는 glGenTexture로 생성한 텍스춰 구분자입니다. 이렇게 텍스춰 구분자를 바인딩해 놓으면 텍스춰 구분자가 현재 활성화된 텍스춰가 됩니다.
* glTexParameteri\(GLenum target, GLenum pname, GLint param\) 현재 활성화된 텍스춰에 다양한 옵션값을 설정합니다. pname은 옵션이름이고 param 은 옵션 값입니다. 설정할 수 있는 옵션의 종류는 다음과 같습니다.
  * GL\_TEXTURE\_MIN\_FILTER
  * GL\_TEXTURE\_MAG\_FILTER
  * GL\_TEXTURE\_WRAP\_S
  * GL\_TEXTURE\_WRAP\_T
  * GL\_GENERATE\_MIPMAP
* glTexImage2D\(GLenum target, GLint level, GLint internalformat, GLsizei width, GLsizei height, GLint border, GLenum format, GLenum type, const GLvoid \* pixels\) 현재 활성화된 텍스춰에 이미지 데이터를 연결하는 함수입니다. 인자가 많은데요. 그리 복잡하지는 않습니다.
  * target : GL\_TEXTURE\_2D만 지원하므로 GL\_TEXTURE\_2D로 설정합니다.
  * level : 디테일 단계인데요. 우선 0으로 설정합니다.
  * internalformat : 텍스춰의 색상 형식인데요. 이미지의 포맷하고 같아야 합니다. 저희는 RGBA만 사용하기로 했으므로 GL\_RGBA로 설정합니다.
  * width : 텍스춰 이미지의 너비입니다. 반드시 2^n 이 되어야 합니다.
  * height : 텍스춰 이미지의 높이입니다. 반드시 2^n 이 되어야 합니다.
  * border : 0으로 설정합니다.
  * format : 이미지의 픽셀 형식입니다. GL\_RGBA로 설정합니다.
  * type : 이미지를 구성하는 픽셀 데이터의 데이터 타입입니다. 우린 GLubyte로 이미지 데이터를 뽑았으니 GL\_UNSIGNED\_BYTE로 설정합니다.
  * pixels : 이미지의 데이터입니다. 간략하게 함수에 대해서 알아보았습니다. 부족한 부분은 OpenGL\|ES의 레퍼런스를 참고해 주시길 바랍니다. 이제 텍스춰를 그리는 코드를 작성해 볼까요?

우선 OGLView를 상속받는 TextureMappedPolygonView 를 생성합니다. 그리고 아래와 같이 인터페이스를 작성합니다.

```objectivec
#import <UIKit/UIKit.h>  
#import "OGLView.h"  
#import "OGLTexture.h"  
@interface TextureMappedPolygonView : OGLView  
{  
    OGLTexture *texture;  
}  
@end
```

텍스춰를 하나만 출력할 것이라서 OGLTexture를 하나만 선언했습니다. 그리고 아래와 같이 정점과 텍스춰좌표\(UV좌표\)를 같이 담는 정점배열을 선언합니다.

```objectivec
GLfloat verticesForGL_TRIANGLE_STRIP[] = {  
    0.2, 0.8, 0.0,          //v1  
    0.0, 1.0,               //UV1  

    0.2, 0.2, 0.0,          //v2  
    0.0, 0.0,               //UV2  

    0.8, 0.8, 0.0,          //v3  
    1.0, 1.0,               //UV3  

    0.8, 0.2, 0.0,          //v4  
    1.0, 0.0,               //UV4  
};
```

그리고 텍스춰를 생성합니다. gl.png는 폴리곤에 맵핑할 그림파일입니다.

![](../../.gitbook/assets/gl.png)



```objectivec
-(void)setupView  
{  
    //: 행렬 모드는 투영 행렬로 변경한다  
    glMatrixMode(GL_PROJECTION);  

    //: 투영행렬을 초기화 한다  
    glLoadIdentity();  

    //: 직교투영으로 설정한다  
    glOrthof(0.0f, 1.0f, 0.0f, 1.0f, -1.0f, 1.0f);  

    //: 뷰포트의 크기를 전체 화면으로 설정한다.  
    glViewport(0, 0, self.bounds.size.width, self.bounds.size.height);  

    //: 텍스춰를 만든다  
    NSString *texturePath = [[NSBundle mainBundle] pathForResource:@"gl" ofType:@"png"];  
    texture = [OGLTexture textureWithImagePath:texturePath];  
    [texture retain];  
}
```

이제 마지막으로 renderView를 아래와 같이 수정합니다.

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
    glVertexPointer(3, GL_FLOAT, sizeof(GLfloat)*5,  
                    verticesForGL_TRIANGLE_STRIP);  
    //: 텍스춰배열을 설정한다  
    glTexCoordPointer(2, GL_FLOAT, sizeof(GLfloat)*5,  
                      &verticesForGL_TRIANGLE_STRIP[0]+3);  

    //: 텍스춰 배열 사용을 ON  
    glEnableClientState(GL_TEXTURE_COORD_ARRAY);  
    //: 정점 배열 사용을 ON  
    glEnableClientState(GL_VERTEX_ARRAY);  
    {  
        //: 처리할 정점의 개수는 4개  
        glEnable(GL_TEXTURE_2D);  
        [texture bindTexture];  
        glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);  
        glDisable(GL_TEXTURE_2D);  
    }  
    //: 정점 배열 사용을 OFF  
    glDisableClientState(GL_VERTEX_ARRAY);  
    //: 텍스춰 좌표 배열 사용을 OFF  
    glDisableClientState(GL_TEXTURE_COORD_ARRAY);  
}
```

그러면 아래와 같이 텍스춰가 폴리곤에 맵핑되어 출력됩니다.

![](../../.gitbook/assets/tut01-1.png)

그런데 그림을 보면 그림이 뒤집혀져 있는 것을 볼 수 있습니다. 그것은 일반이미지를 OpenGL\|ES에 맵핑할 때 생기는 현상입니다. PVRTC 압축 이미지는 제대로 출력되지만 우리가 사용한 png파일은 PVRTC 압축파일이 아니기 때문에 OGLTexture의 loadImage에 다음과 같이 CGBitmapContext를 뒤집는 코드를 작성해야 합니다.

```objectivec
-(BOOL)loadUIImage:(NSString *)path  
{  
    UIImage *image = [UIImage imageWithContentsOfFile:path];  
    if(image == nil)  
        return NO;  

    //: 이미지의 정보를 얻어 온다  
    CGImageRef cgImage = [image CGImage];  
    width = CGImageGetWidth(cgImage);  
    height= CGImageGetHeight(cgImage);  

    //: 이미지의 너비와 높이가 2의 승수인지 살펴봐야 하지만  
    //: 생략한다  

    //: 이미지 데이터의 바이트 배열을 만들기 위해서  
    //: 비트맵 컨텍스트를 만들고 거기에 이미지를 그린다  
    //: 텍스춰로 사용하는 이미지는 모두 RGBA 4바이트로 가정한다  
    if(imageData)  
        free(imageData);  

    imageData = (GLubyte *)calloc(width*height*4, sizeof(GLubyte));  
    if(imageData == NULL)  
        return NO;  

    //: 이미지를 그릴 비트맵컨텍스트를 생성한다  
    CGContextRef bitmapContext = CGBitmapContextCreate(imageData,  
                                                       width,  
                                                       height,  
                                                       8,  
                                                       width*4,  
                                                       CGImageGetColorSpace(cgImage),  
                                                       kCGImageAlphaPremultipliedLast);  

    //: PVRTC 압축 이미지가 아니면 텍스춰이미지가 역상으로 나온다  
    //: 그것을 바로 잡아 준다  
    CGContextTranslateCTM (bitmapContext, 0, height);  
    CGContextScaleCTM (bitmapContext, 1.0, -1.0);  

    //: 이미지를 비트맵컨텍스트에 그린다  
    CGContextDrawImage(bitmapContext,  
                       CGRectMake(0, 0, width, height),  
                       cgImage);  

    //: 비트맵데이터만 필요하므로 컨텍스트는 메모리 해제한다  
    CGContextRelease(bitmapContext);  
    return YES;  
}
```

이제 다시 실행하면 아래와 같이 텍스처가 제대로 출력되는 것을 확인할 수 있습니다.

![](../../.gitbook/assets/tut02-1.png)

그림만 출력하면 심심하니깐 정점에 색상도 같이 출력해 보겠습니다.

```objectivec
GLfloat verticesForGL_TRIANGLE_STRIP[] = {  
    0.2, 0.8, 0.0,          //v1  
    1.0, 0.0, 0.0, 1.0,     //R  
    0.0, 1.0,               //UV1  

    0.2, 0.2, 0.0,          //v2  
    0.0, 1.0, 0.0, 1.0,     //G  
    0.0, 0.0,               //UV2  

    0.8, 0.8, 0.0,          //v3  
    0.0, 0.0, 1.0, 1.0,     //B  
    1.0, 1.0,               //UV3  

    0.8, 0.2, 0.0,          //v4  
    1.0, 1.0, 0.0, 1.0,     //Y  
    1.0, 0.0,               //UV4  
};  

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
    glVertexPointer(3, GL_FLOAT, sizeof(GLfloat)*9,  
                    verticesForGL_TRIANGLE_STRIP);  
    //: 색상배열을 설정한다  
    glColorPointer(4, GL_FLOAT, sizeof(GLfloat)*9,  
                    &verticesForGL_TRIANGLE_STRIP[0]+3);  
    //: 텍스춰배열을 설정한다ㅏ  
    glTexCoordPointer(2, GL_FLOAT, sizeof(GLfloat)*9,  
                      &verticesForGL_TRIANGLE_STRIP[0]+7);  

    //: 색상 칠하기 방법을 설정한다  
    glShadeModel(GL_SMOOTH);  

    //: 텍스춰 배열 사용을 ON  
    glEnableClientState(GL_TEXTURE_COORD_ARRAY);  
    //: 색상 배열 사용을 ON  
    glEnableClientState(GL_COLOR_ARRAY);  
    //: 정점 배열 사용을 ON  
    glEnableClientState(GL_VERTEX_ARRAY);  
    {  
        //: 처리할 정점의 개수는 4개  
        glEnable(GL_TEXTURE_2D);  
        [texture bindTexture];  
        glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);  
        glDisable(GL_TEXTURE_2D);  
    }  
    //: 정점 배열 사용을 OFF  
    glDisableClientState(GL_VERTEX_ARRAY);  
    //: 색상 배열 사용을 OFF  
    glDisableClientState(GL_COLOR_ARRAY);  
    //: 텍스춰 좌표 배열 사용을 OFF  
    glDisableClientState(GL_TEXTURE_COORD_ARRAY);  
}
```

![](../../.gitbook/assets/tut03%20%282%29.png)

여기서 이번 강좌를 마치고 다음 튜토리얼에서 텍스춰의 UV좌표를 다뤄 보겠습니다. 감사합니다. :\)

