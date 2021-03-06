# MiniLibX

이 문서는 아래 링크의 글의 번역 버전입니다. \
https://harm-smits.github.io/42docs/libs/minilibx/introduction.html

### Introduction





1.  들어가면서

    MiniLibx 는 작은 그래픽 라이브러리로 X-window 그리고 Cocoa에 대한 지식 없이도 스크린 상에 무언가를 렌더링 하는 가장 기본적인 일을 할 수 있는 라이브러리다. 이 라이브러리는 소위 simple windo creation, questionable drawing tool, half-ass image functions and a weird event를 갖고 있다(ㅈㄴ 안좋다는 뜻같다...)
2.  X-Window 란

    X-Window 란 Unix 를 위한 네트워크 지향형 그래픽 시스템이다. 예를 들어 원격 데스크톱을 연결을 할 대 사용된다. 이러한 실행의 대표적 예로는 TeamViewr가 있겠다.
3.  맥 OS 를 관에

    macOS는 그것의 화면에 그래픽 접근을 처리한다. 하지만 이에 접속하기 위해서 우리는 반드시 우리의 어플리케이션을 스크린을 조작하고, 창 시스템, 키보드와 마우스를 다루는 하부 맥OS의 그래픽 프레임워크에 등록되어야 한다.

### Getting started

1.  들어가면서

    이제 MiniLibX가 무엇을 하는지를 알았고, 우리는 매우 기초적인 것을 진행할 것입니다. 이것들은 이 라이브러리를 사용하여 성능적으로 좋은 코드를 어떻게 작성하는지에 대한 명확한 이해를 제공할 것입니다. 다양한 프로젝트를 위해, 퍼포먼스는 핵심입니다. 이 섹션을 철저하게 읽어내는 것은 정말 중요합니다.
2. 설치 절차
   *   Compilation on mac OS

       MiniLibX는 맥 OS의 Cocoa(Appkit) 와 OpenGL(더이상 X11을 사용하지 않는다) 필요로 하기 때문에, 우리는 그것들을 링킹할 필요가 있습니다. 이는 복잡한 컴파일 과정일 수 있습니다. 기본적인 컴파일 절차는 다음과 같습니다.

       목적파일을 위해, 당신의 프로젝트 루트 단에 `mlx`로 지정된 디렉토리 안에 mlx 소스코드가 있다는 전제 하에 다음 규칙을 당신의 Makefile에 추가하십시오.

       ```makefile
       %.o: %.c
       	$(CC) -Wall -Wextra -Werror -Imlx -c $< -o $@
       ```

       필요한 내장 맥 OS API를 연결시키기 위해

       ```makefile
       $(NAME): $(OBJ)
       	$(CC) $(OBJ) -Lmlx -lmlx -framework OpenGL -framework AppKit -o $(NAME)
       ```

       동적 라이브러리 `libmlx.dylib`가 당신의 빌드 타겟과 동일한 디렉토리에 있어야 함을 명심하십시오.

       💡 해당 방법은 x86에 해당 하는 openGL 기준의 설명이다. mms 버전에 대해선 다르게 되어야 하며 고민이 필요하다.
   *   Compilation on Linux

       리눅스의 경우, 당신은 리눅스 호환의 MLX 버전인 [\[이 링크\]](https://github.com/42Paris/minilibx-linux)를 사용할 수 있다. 해당 함수들은 정확히 동일하고, 동일한 함수 호출을 사용한다. 기억해야 할 것은, 객체 구현이 아키텍처에 따라 다르기 때문에 이미지 상에서 memory magic 을 사용하는 것은 다를 수 있다. 다음으로 새로운 폴더에 리눅스용 MLX 를 압축 해제한다. 프로젝트 디렉토리 상에서 `mlx_linux` 라고 지정하면 된다.

       MiniLibX for Linux 는 `xorg`, `x11`, `zlib` 가 필요시 된다. 그러므로 다음 의존성 패키지를 설치할 필요가 있다.

       `xorg` `libext-dev` `zlib1g-dev` 해당 의존성 패키지를 다음을 통해 설치 가능하다.

       ```bash
       sudo apt-get update && sudo apt-get install xorg libxext-dev zlib1g-dev libbsd-dev
       ```

       이제 남은 것은 MLX 의 설정이다. `configure`스크립트를 제공된 레포지터리 루트에서 실행하면 된다.

       목적 파일을 위해, 당신의 프로젝트 루트에 `mlx_linux` 라고 명명한 디렉토리에 mlx 소스코드를 갖고 있다는 전제 하에 당신의 Makefile 에 다음 룰을 추가하면 된다.

       ```makefile
       $(NAME): $(OBJ)
       	$(CC) $(OBJ) -Lmlx_linux -lmlx_Linux -L/usr/lib -Imlx_linux -lXext -lX11 -lm -lz -o $(NAME)
       ```
   * Getting a screen on Windows 10(WSL2)
   * Getting a screen on Windows 11 (WSLg)
3.  Initialization

    MiniLibX 라이브러리를 사용해 뭔가를 해보기 전에, 모든 함수에 접근하기 위해 우리는 반드시 `<mlx.h>` 를 사용해야 하며, 우선 `mlx_init` 함수를 실행해야 합니다. 이 함수는 적절한 그래픽 시스템과의 연결을 성립하게 만들며, 우리의 최근에 MLX 인스턴스의 지점을 쥔 `void *` 를 반환합니다.

    ```c
    #include <mlx.h>

    int	main(void)
    {
    	void	*mlx;

    	mlx = mlx_init();
    }
    ```

    해당 코드를 실행하게 되면, 아무것도 팝업되지 않으며, 어떤 것도 렌더링 되지 않음을 알 수 있습니다. 이것은 윈도우를 만들어내지 않았기 때문입니다. 그래서 작은 창 하나를 초기화하고 영원히 띄워 보겠습니다. 해당 프로그램이 완성되면 검은 창이 뜨게 될 것입니다. `CTRL+C` 로 종료할 수 있습니다. 이를 위해 우리는 `mlx_new_windw` 함수를 간단하게 불러오면 됩니다. 해당 함수는 생성했던 창의 포인터를 반환 할 것입니다. 여기엔 창의 가로, 높이, 그리고 타이틀을 입력하는 것이 가능합니다. 그 뒤엔 `mlx_loop` 를 통해 창의 렌더링을 실행시킬 수 있습니다.

    가로 1920, 세로 1080에 이름이 “Hello world!” 인 창을 생성해봅시다.

    ```c
    #include <mlx.h>

    int	main(void)
    {
    	void	*mlx;
    	void	*mlx_win;

    	mlx = mlx_init();
    	mlx_win = mlx_new_window(mlx, 1920, 1080, "Hello world!");
    	mlx_loop(mlx);
    }
    ```
4.  Writing pixels to a image

    이제 기초 창 관리를 할 수 있으니, 창에 pixel들을 밀어넣어 봅시다. 어떻게 이런 픽셀들을 취할지는 당신에게 달려있지만, 몇가지 최적화된 방법에 대해 논의 해봅시다. 우선, `mlx_pixel_put` 함수가 매우, 매우 느리다는 점을 설명해야 합니다. 이것은 이 함수가 창에 즉시 픽셀을 push 하려고 시도하기 때문입니다.(전체 렌더 되는 프레임을 위해 기다리는 행위 없이 진행되기 때문입니다.) 이 한 가지 이유로 인하여, 우리의 픽셀 전체를 이미지로 버퍼링을 해야 합니다. 그 뒤에 우리는 이를 창 안에 push 해야 합니다. 상당히 복잡하게 들리지만, 이는 그렇게 어렵지 않습니다.

    우선, mlx가 요구하는 이미지의 형태가 무엇인지를 이해하는걸로 시작해야 합니다. 만약 이미지를 개시하면, 중요한 변수 몇개를 쓸 몇 가지 포인터를 전달해야 합니다. 첫 번째는 `bpp` 인데, 이는 다른 말로 bits per pixel 입니다. 픽셀들은 기본적으로 정수들이며, 이들은 대게 4바이트입니다. 하지만 small endian으로 처리해준다면 달라질 수 있습니다.(이는 원격 디스플레이나 8비트 색상만 가지는 경우를 의미합니다.)

    이제 image 를 1920x 1080 의 해상도로 이미지를 실행해볼 수 있습니다.

    ```c
    #include <mlx.h>

    int	main(void)
    {
    	void	*img;
    	void	*mlx;

    	mlx = mlx_init();
    	img = mlx_new_image(mlx, 1920, 1080);
    }
    ```

    어렵지 않죠? 이제 이미지를 취했습니다. 하지만 어떻게 정확하게 이것에 pixel들을 쓸까요? 이를 위해선 우리는 그에따라 바이트를 변경할 메모리 주소를 취할 필요가 있습니다. 이 주소를 우리는 다음과 같이 되찾아 올 수 있습니다.

    ```c
    #include <mlx.h>

    typedef struct	s_data {
    	void	*img;
    	char	*addr;
    	int		bits_per_pixel;
    	int		line_length;
    	int		endian;
    }				t_data;

    int	main(void)
    {
    	void	*mlx;
    	t_data	img;

    	mlx = mlx_init();
    	img.img = mlx_new_image(mlx, 1920, 1080);

    	/*
    	** 이미지를 작성한 후, 우리는 `mlx_get_data_addr` 을 호출 할 수 있습니다. 
    	** `bits_per_pixel`, `line_length`, `endian` 을 레퍼런스로 전달합니다. 
    	** 최신 데이터 주소로 세팅 될 것입니다. 
    	*/
    	img.addr = mlx_get_data_addr(img.img, &img.bits_per_pixel, &img.line_length,
    								&img.endian);
    }
    ```

    어떻게 우리가 `bits_per_pixel`, `line_length`, `endian` 변수를 레퍼런스로 어떻게 보내는지 알겠습니까? 위에 기제 된 것처럼 MiniLibX에 의해 세팅됩니다.

    이제 우리는 이미지 주소를 알게 되었습니다. 하지만 여전히 픽셀들이 들어있진 않죠. 시작 전에, 우리는 바이트들이 정렬이 되어 있지 않음을 이해해야 합니다. 이것은 `line_length` 가 실제 창 너비와 다름을 의미합니다. 그러므로 우리는 **항상** `mlx_get_data_addr` 함수에 의해 설정된 line length를 사용하는 메모리의 offset을 계산해야 합니다.

    이를 다음 공식을 이용하면 매우 쉽게 계산이 가능합니다.

    ```c
    int offset = (y * line_length + x * (bits_per_pixel / 8));
    ```

    이제 우리는 작성하는 곳을 알기에, `mlx_pixel_put` 함수의 동작을 모사하는 함수를 매우 쉽게 작성할 수 있게 되었습니다. 물론 몇배는 더 빠르게 말이죠.

    ```c
    typedef struct	s_data {
    	void	*img;
    	char	*addr;
    	int		bits_per_pixel;
    	int		line_length;
    	int		endian;
    }				t_data;

    void	my_mlx_pixel_put(t_data *data, int x, int y, int color)
    {
    	char	*dst;

    	dst = data->addr + (y * data->line_length + x * (data->bits_per_pixel / 8));
    	*(unsigned int*)dst = color;
    }
    ```

    이제 이슈를 야기 시킬겁니다. 창 안에서 실시간으로 이미지가 표지되기 때문에, 이것을 쓰기 도중에는 수 많은 화면 티어링을 같은 이미지를 바꾸는 것이 야기시키기 때문입니다. 그러므로 순간적인 프레임을 유지하기 위해 2개 혹은 그 이상의 이미지를 작성할 필요가 있습니다. 그 다음에 임시의 이미지를 작성할 수 있고, 그 결과 최근에 표시한 이미지에 쓸 필요가 없어집니다.
5.  Pushing image to a window

    이제 마침내 우리의 이미지를 만들어낼 수 있고, 창에 이것들을 push 해야 합니다. 이것은 상당히 직관적입니다. 빨간 필셀을 (5, 5) 지점에 어떻게 넣어지는지를 관찰해봅시다.

    ```c
    #include <mlx.h>

    typedef struct	s_data {
    	void	*img;
    	char	*addr;
    	int		bits_per_pixel;
    	int		line_length;
    	int		endian;
    }				t_data;

    int	main(void)
    {
    	void	*mlx;
    	void	*mlx_win;
    	t_data	img;

    	mlx = mlx_init();
    	mlx_win = mlx_new_window(mlx, 1920, 1080, "Hello world!");
    	img.img = mlx_new_image(mlx, 1920, 1080);
    	img.addr = mlx_get_data_addr(img.img, &img.bits_per_pixel, &img.line_length,
    								&img.endian);
    	my_mlx_pixel_put(&img, 5, 5, 0x00FF0000);
    	mlx_put_image_to_window(mlx, mlx_win, img.img, 0, 0);
    	mlx_loop(mlx);
    }
    /*
    0x00FF0000 는 16진법 ARGB(0,255,0,0)를 표현한 것입니다. 
    */
    ```
6.  Test Your Skills!

    기초를 이해한 당신이라면, 라이브러리와 함께 친숙해지고 약간 신이 날 것입니다. 테스트 한번 해보시길!

    * 사각형, 원, 삼각형, 육각형을 스크린 상에 pixel들로 그려봅시다.
    * 그라디에션을 추가하길 시도해보고, 무지개를 그려보고, rgb 색깔 사용에 친숙해 져보십시오.
    * 반복문을 활용해서 이미지를 생성하는 것으로 텍스쳐를 만들어봅시다.

    #### Colors

### Colors

색은 정수 형태로 표현됩니다. 그러므로 몇 가지 ARGB 값을 포함할 수 있는 정수를 획득할 수 있도록 기믹적인 것들을 필요로 합니다.

1.  The color integer standard

    | 문자 | 뜻        |
    | -- | -------- |
    | T  | 투명도      |
    | R  | 붉은색 컴포넌트 |
    | G  | 초록색 컴포넌트 |
    | B  | 파란색 컴포넌트 |

    RGB 색은 다음처럼 초기화 될 수 있다.

    | 색     | TRGB 표시    |
    | ----- | ---------- |
    | red   | 0x00FF0000 |
    | green | 0x0000FF00 |
    | blue  | 0x000000FF |
2.  Encoding and decoding colors

    색을 encode, decode 하는 방법으로 두가지를 사용할 수 있다.

    * BitShifting
    * char/int conversion
    *   BitShifting

        각 바이트는 `2^8 = 256` 의 값(1 byte = 8 bits)을 지닌다. 그리고 RGB 값의 범위는 0 \~ 255를 가진다. 이는 정수 값 안에 완벽하게 담을 수 있다. 프로그램적으로 값들을 담기 위하여, 우리는 `bitshifting` 을 사용 한다. 이제 정확하게 만들어내는 함수를 만들어봅시다.

        ```c
        int creat_trgb(int t, int r, int g, int b)
        {
        	return (t << 24 | r << 16 | g << 8 | b);
        }
        ```

        정수는 오른쪽부터 왼쪽으로 정렬되므로, 비트들을 거꾸로의 값에 따라 각 값을 bitshift 할 필요가 있습니다. 더불어 완전히 정 반대로 하는 것 그리고 인코딩된 TRGB로부터 정수 값을 복구 하는 것도 가능합니다.

        ```c
        int	get_t(int trgb)
        {
        	return ((trgb >> 24) & 0xFF); // 1바이트를 모두 1로 채운 값과 & 연산으로 1인 경우만 남겨버리게 된다. 
        }

        int	get_r(int trgb)
        {
        	return ((trgb >> 16) & 0xFF);
        }

        int	get_g(int trgb)
        {
        	return ((trgb >> 8) & 0xFF);
        }

        int	get_b(int trgb)
        {
        	return (trgb & 0xFF);
        }
        ```
    *   char/int conversion

        1 바이트 당 RGB 값이 지정되고, 당연히 16진수의 정수값으로 지정한 것을 반대로 설정하는 것도 가능 합니다. 이를 위한 컨버팅은 아래와 같은 방식으로 할 수도 있습니다. 아래는

        ```c
        int	create_trgb(unsigned char t, unsigned char r, unsigned char g, unsigned char b)
        {
        	return (*(int *)(unsigned char [4]){b, g, r, t}); //들어오는 문자열로 된 정수값 1바이트(비부호형) 을 묶어서 배열로 만들고 다시 정수로 형변환을 하여 정수값으로 환산한 형태
        }

        unsigned char	get_t(int trgb)
        {
        	return (((unsigned char *)&trgb)[3]); //받은 정수를 다시 비부호형 char 타입으로 형변환 한뒤 배열처럼 하여 해당 위치에 맞는 정수값을 출력하는 형식 
        }

        unsigned char	get_r(int trgb)
        {
        	return (((unsigned char *)&trgb)[2]);
        }

        unsigned char	get_g(int trgb)
        {
        	return (((unsigned char *)&trgb)[1]);
        }

        unsigned char	get_b(int trgb)
        {
        	return (((unsigned char *)&trgb)[0]);
        }
        ```

        변환을 이해하기 위해선 아래의 표를 참고하시면 됩니다. `0x0FAE1` 는 `int trgb` 의 변수 주소다.

        | Address | char            | int          |
        | ------- | --------------- | ------------ |
        | 0x0FAE1 | unsigned char b | int trgb     |
        | 0x0FAE2 | unsigned char g | \[allocated] |
        | 0x0FAE3 | unsigned char r | \[allocated] |
        | 0x0FAE4 | unsigned char t | \[allocated] |
3.  Test your skills

    이제 색을 어떤 식으로 초기화 할 수 있는지의 기초를 이해했다면, 익숙해지고, 다음 컬러 조작 함수들을 만들어보는 것을 시도하면 된다.

    * `add_shade` 는 double(거리)와 int(색) 을 인자대로 수용해주는 함수이다. 0은 색상에 쉐이딩을 추가 하지 않으며, 반면에 1은 완벽하게 어둡게 만들어 줍니다. 0.5 는 절반정도 어둡게 만들고, .25는 1/4 정도 그렇게 만듭니다. 해당 포인트를 얻어 보세요.
    * `get_opposite` 함수는 int(색) 인자를 수용하고, 생상을 반전 시켜 줍니다. ⇒ 보색 활용해야함

### Hooks

컴퓨터 프로그래밍에서 hooking 이란 용어는 SW 컴포넌트 사이에서 지나가는 메시지, 혹은 이벤트들, 혹은 함수 호출을 가로채감으로써 다른 SW 컴포넌트의 혹은 어플리케이션의, 혹은 운영체제의 행동, 인자를 대체하는데 사용하는 기술을 아우르는 말이다. 그러한 함수 콜, 이벤트들, 메시지를 가로채는 걸 다루는 code 들을 hook 이라고 부른다.

1.  Introduction

    Hooking은 디버깅이라던지, 함수의 확장 등을 포함하여 다양한 목적으로 사용된다. 예시로서는 키보드를 가로채거나, 어플리케이션에 도달하기 이전에 마우스 이벤트 메시지를 취한다거나, 특정 어플리케이션이나 다른 컴포넌트의 함수 수정 혹은 감시하는 행위를 위해 운영체제의 호출을 가로채는 등을 들 수 있다. 이는 벤치마킹 프로그램등에서도 널리 쓰이며, 이는 3D 게임의 프레임레이트를 측정하는 용으로 쓸 수도 있다.

    그러므로 mlx의 근간에 hooking이 있는 것은 전혀 이상한게 아니다.
2.  Hooking into key events

    hooking 은 아마도 어려워 보이지만, 실제론 간단하게 구현 가능하다.

    ```c
    #include <mlx.h>
    #include <stdio.h>

    typedef struct	s_vars {
    	void	*mlx;
    	void	*win;
    }				t_vars;

    int	key_hook(int keycode, t_vars *vars)
    {
    	printf("Hello from key_hook!\n");
    	return (0);
    }

    int	main(void)
    {
    	t_vars	vars;

    	vars.mlx = mlx_init();
    	vars.win = mlx_new_window(vars.mlx, 640, 480, "Hello world!");
    	mlx_key_hook(vars.win, key_hook, &vars);
    	mlx_loop(vars.mlx);
    }
    ```

    이제 키를 누를 때 메시지를 출력하는 함수를 등록했다. 보시다시피, `mlx_key_hook` 를 사용해서 등록이 가능하다. 그러나 간단한 함수 `mlx_hook` 를 X11 이벤트 타입을 적절히 활용한다면, 백그라운드에서 간단하게 함수를 hook 하는 것이 가능하다.
3.  Hooking into mouse events

    마우스 이벤트 역시 hook 하는 것이 가능하다.

    ![mouse](./src/Untitled.png)

    Mac OS 상에 마우스 코드는 다음과 같다.

    * Left click: 1
    * Right click: 2
    * Middle click: 3
    * Scroll up: 4
    * Scroll down : 5
4.  Test your skills!

    우리걸로 만들기 위해 시도해볼것!

    1. 키를 누르면 키코드가 터미널 상에 출력되도록 해보기
    2. 마우스를 움직이면 현재의 마우스가 있는 좌표를 출력해보기
    3. 마우스 버튼이 눌렀을 누르면 창위에서 터미널로 움직인 각을 출력해보시오

### Events

1.  Introduction

    이벤트들은 mlx 안에서 대화식 어플리케이션 작성의 토대이다. 그러므로 차후의 그래픽 프로젝트에서도 사용될 것이므로 이 챕터를 완벽하게 이해하는 것은 필수라고 할 수 있다.

    모든 mlx 의 hook 들은 이벤트가 발생할 때마다 호출되어지는 함수 그 이상도 그 이하도 아니다. 모든 이벤트 함수들을 완벽하게 숙지하는 것은 필수는 아니지만, 각 X11 이벤트들에 따라서 빠르게 검토해볼 필요는 있다.

    1. MacOS version 주) Mac OS 에서 Cocoa(Appkit) 그리고 OpenGL 버전은 부분적으로 X11 이벤트를 지원하지만, X11 mask는 지원하지 않는다(mlx의 x\_mask 인자는 쓸모가 없다. 항상 0으로 유지 시켜라).
    2.  지원 이벤트들 :

        ```c
        enum {
        	ON_KEYDOWN = 2,
        	ON_KEYUP = 3,
        	ON_MOUSEDOWN = 4,
        	ON_MOUSEUP = 5,
        	ON_MOUSEMOVE = 6,
        	ON_EXPOSE = 12,
        	ON_DESTROY = 17
        };

        // usage:
        mlx_hook(vars.win, ON_DESTROY, 0, close, &vars); 
        ```
2.  X11 interface

    X11 은 mlx와 함께 사용되는 라이브러리 입니다. 그러므로 mlx의 모든 이벤트를 확인하는데 매우 유용한 헤더임에 틀림 없습니다.

    1.  X11 events

        다양한 이벤트 들이 존재합니다.

        | Key | Event          | Key | Event            | Key | Event            |
        | --- | -------------- | --- | ---------------- | --- | ---------------- |
        | 02  | KeyPess        | 14  | NoExpose         | 26  | CirculateNotify  |
        | 03  | KeyRelease     | 15  | VisibilityNotify | 27  | CirculateRequest |
        | 04  | ButtonPress    | 16  | CreateNotify     | 28  | PropertyNotify   |
        | 05  | ButtonRelease  | 17  | DestroyNotify    | 29  | SelectionClear   |
        | 06  | MotionNotify   | 18  | UnmapNotify      | 30  | SelectionRequest |
        | 07  | EnterNotify    | 19  | MapNotify        | 31  | SelectionNotify  |
        | 08  | LeaveNotify    | 20  | MapRequest       | 32  | ColormapNotify   |
        | 09  | FocusIn        | 21  | ReparentNotify   | 33  | ClientMessage    |
        | 10  | FocusOut       | 22  | ConfigureNotify  | 34  | MappingNotify    |
        | 11  | KeymapNotify   | 23  | ConfigureRequest | 35  | GenericEvent     |
        | 12  | Expose         | 24  | GravityNotify    | 36  | LASTEvent        |
        | 13  | GraphicsExpose | 25  | ResizeRequest    |     |                  |

        만약 몇몇 이벤트들이 무엇인지 알기 어려울 수 있지만, 걱정마세요. 아마 그게 필요하진 않을 겁니다. 만약 그렇다면 이 문서를 읽으시면 됩니다. [\[링크\]](https://tronche.com/gui/x/xlib/events/)
    2.  X11 masks

        각 X11 이벤트 들은 또한 해당하는 mask도 있습니다. 이 방식은 당신이 누른 한키에 대해 등록할 수 있거나, 혹은 마스크를 기본으로 두는 경우 모든 키를 등록하는 것이 가능합니다. 그러므로 key mask 는 당신의 이벤트 구독 상태로부터 이벤트들을 화이트리스트, 블랙리스트화 하는 것을 허락하게 만들어줍니다. 아래가 할당된 마스크 들입니다.

        | Mask     | Description           |   | Mask     | Description              |
        | -------- | --------------------- | - | -------- | ------------------------ |
        | 0L       | NoEventMask           |   | (1L<<12) | Button5MotionMask        |
        | (1L<<0)  | KeyPressMask          |   | (1L<<13) | ButtonMotionMask         |
        | (1L<<1)  | KeyReleaseMask        |   | (1L<<14) | KeymapStateMask          |
        | (1L<<2)  | ButtonPressMask       |   | (1L<<15) | ExposureMask             |
        | (1L<<3)  | ButtonReleaseMask     |   | (1L<<16) | VisibilityChangeMask     |
        | (1L<<4)  | EnterWindowMask       |   | (1L<<17) | StructureNotifyMask      |
        | (1L<<5)  | LeaveWindowMask       |   | (1L<<18) | ResizeRedirectMask       |
        | (1L<<6)  | PointerMotionMask     |   | (1L<<19) | SubstructureNotifyMask   |
        | (1L<<7)  | PointerMotionHintMask |   | (1L<<20) | SubstructureRedirectMask |
        | (1L<<8)  | Button1MotionMask     |   | (1L<<21) | FocusChangeMask          |
        | (1L<<9)  | Button2MotionMask     |   | (1L<<22) | PropertyChangeMask       |
        | (1L<<10) | Button3MotionMask     |   | (1L<<23) | ColormapChangeMask       |
        | (1L<<11) | Button4MotionMask     |   | (1L<<24) | OwnerGrabButtonMask      |
3. Hooking into events
   1.  mlx\_hook

       이벤트들로부터 hooking 은 mlx 가 제공하는 가장 강력한 도구 중에 하나입니다. 단순한 훜 등록 함수를 호출함과 함께 앞서 언급한 이벤트들의 종류를 등록하는걸 돕습니다ㅏ.

       이걸 위해, 우리는 `mlx_hook` 함수를 호출합니다.

       ```c
       void mlx_hook(mlx_win_list_t *win_ptr, int x_event, int x_mask, int (*f)(), void *param)
       ```

       주의) 몇몇 mlx 버전에 따라 `x_mask` 를 수행하지 않고, 무슨 값이든지간에 마스크가 존재하지 않을 수 잇습니다.
   2.  Prototype of event functions

       | Hooking event   | code | Prototype                                         |
       | --------------- | ---- | ------------------------------------------------- |
       | ON\_KEYDOWN     | 2    | int (\*f)(int keycode, void \*param)              |
       | ON\_KEYUP\*     | 3    | int (\*f)(int keycode, void \*param)              |
       | ON\_MOUSEDOWN\* | 4    | int (\*f)(int button, int x, int y, void \*param) |
       | ON\_MOUSEUP     | 5    | int (\*f)(int button, int x, int y, void \*param) |
       | ON\_MOUSEMOVE   | 6    | int (\*f)(int x, int y, void \*param)             |
       | ON\_EXPOSE\*    | 12   | int (\*f)(void \*param)                           |
       | ON\_DESTROY     | 17   | int (\*f)(void \*param)                           |

       * 는 mlx\_hook를 앨리아스 이다.
   3.  Hooking alias

       mlx api 는 몇가지 앨리아스 hooking 함수를 갖고 있다.

       * `mlx_expose_hook` : 노출 이벤트(12) 에 대한 mlx\_hook 의 앨리아스 함수이다.
       * `mlx_key_hook` : key up 이벤트(3)에 대한 mlx\_hook의 앨리아스 함수이다.
       * `mlx_mouse_hook` : 마우스 다운 이벤트(4) 에 대한 mlx\_hook의 앨리아스 함수이다.
   4.  Example

       `mlx_key_hook` 의 호출 대신의 예시로 `keypress` 그리고 `keyRelease` 이벤트를 등록할 수 있다. 이제 키를 누르게 되면 창은 닫히게 될 것이다.

       ```c
       #include <mlx.h>

       typedef struct	s_vars {
       	void	*mlx;
       	void	*win;
       }				t_vars;

       int	close(int keycode, t_vars *vars)
       {
       	mlx_destroy_window(vars->mlx, vars->win);
       	return (0);
       }

       int	main(void)
       {
       	t_vars	vars;

       	vars.mlx = mlx_init();
       	vars.win = mlx_new_window(vars.mlx, 1920, 1080, "Hello world!");
       	mlx_hook(vars.win, 2, 1L<<0, close, &vars);
       	mlx_loop(vars.mlx);
       }
       ```
4.  Test your skils!

    이제 뿌연 아이디어를 어떻게 이것을 작동시키게 만들지를 생각해보자. 언제든지 아래의 이벤트 들의 hook handler들을 만들어보자

    * `ESC` 를 누르면, 너의 창은 닫히게 만든다. ✅
    * 창 사이즈가 변경이 되면, 이에 따라 터미널에 무언가 출력되어야 한다.
    * 붉은 십자가가 클릭되면, 창을 닫아야 한다.
    * x 초 이상 길게 키를 누르면, 터미널에 무언가 표시되어야 한다.
    * 마우스가 창 안으로 들어가게 되면, 터미널 상에 `Hello!` 라고 나오게 하고, 떠나면 `Bye!` 라고 출력되게 만든다. ✅

### Loops

1.  Introduction

    이제 당신은 마침내 MiniLibX Library의 기초를 이해했습니다. 그러니 창 안에 작은 애니메이션을 그려보는걸 시작합시다. 이걸 위해 우리는 새로운 함수 `mlx_loop` , `mlx_loop_hook` 이라는 새로운 두 함수를 사용해 볼 겁니다.

    반복문은 새로운 프레임들을 렌더링하기 위해 `mlx_loop_hook` 으로 등록된 hook을 호출 이어나가는 MiniLibX의 기능 중 하나입니다.
2.  Hooking into loops

    반복문을 초기화 하기 위해, `mlx_loop` 함수를 `mlx` 인스턴스를 가지고 호출을 합니다.

    ```c
    #include <mlx.h>

    int	main(void)
    {
    	void	*mlx;

    	mlx = mlx_init();
    	mlx_loop(mlx);
    }
    ```

    우리가 따로 등록한 loop hook가 없으로 아무것도 안 일어날 겁니다. 그러므로 우리는 우리의 프레임에 어떤 것들을 작성 불가능합니다.

    이걸 해내기 위해서, 우리는 getting started 챕터에서 묘사했던 변화를 사용하고, 새로운 창을 만들어야 할 것입니다. 당신의 인자들을 잘 전달한다 할 수 있고, 능숙하다는 전재하에 언급합니다. 화이트보드 버전 같은 그것은 다음같이 표현될 수 있다.

    ```c
    #include <mlx.h>

    int	render_next_frame(void *YourStruct);

    int	main(void)
    {
    	void	*mlx;

    	mlx = mlx_init();
    	mlx_loop_hook(mlx, render_next_frame, YourStruct);
    	mlx_loop(mlx);
    }
    ```

    이제 각 프레임을 위해 `render_next_frame` 이라 불리는 함수와, `YourStruct` 라는 인자를 호출합니다. 이것은 지속적이게 다수의 함수호출이 예상된다. 그러므로 해당 내용을 사용하는 것은 큰 메리트가 된다
3. Test your skills
   * 움직이는 모든 컬러를 가진 무지개를 만들어라.
   * wasd키를 사용하여 당신의 화면 넘어로 움직일 수 있다.

### Images

1.  Introduction

    mlx 상에서 완벽한 잠재력 끌어 내기 위해 중요한 도구중에 하나이다. 이 함수들은 당신이 이미지 객체로부터 파일을 직접 읽는 것을 허락해줍니다. 이는 텍스쳐나 스프라이트 등에 매우 유용합니다.
2.  Reading images

    파일부터 이미지 객체까지 읽어 들이기 위하여, 우리는 XMP 혹은 PNG 포맷을 필요로 합니다. 읽기 위해, 우리는 `mlx_xpm_file_to_image` 와 `mlx_png_file_to_image` 라는 함수 호출이 필요합니다. 이때 `mlx_png_file_to_image`는 현재 메모리 누수가 존재한다. 두 함수들은 모두 동일한 인자들과 동일한 사용성을 제공한다.

    ```c
    #include <mlx.h>

    int	main(void)
    {
    	void	*mlx;
    	void	*img;
    	char	*relative_path = "./test.xpm";
    	int		img_width;
    	int		img_height;

    	mlx = mlx_init();
    	img = mlx_xpm_file_to_image(mlx, relative_path, &img_width, &img_height);
    }
    ```

    만약 `img` 변수가 널값이라면, 이는 이미지를 읽는 것에 실패했다는 의미다. 또한 `img_width` 와 `img_height` 은 스프라이트를 읽을 때도 동일한 존재다.
3. Test your skills!
   1. 기호에 맞게 커서를 Import 해보고, 창 안을 자연스럽게 돌아다녀 봅시다.
   2. 텍스쳐를 import 하고 복사해서 창 전체에 복사해봅시다.

### Sync

1.  What is sync?

    이전에 설명한 것처럼, 직접 mlx 를 가지고 프레임을 관리할수 있습니다. 하지만 이는 꽤나 끔직하며 시간 소비적인 일입니다. 게다가 더 많은 메모리를 먹고, 우리가 관리하는 프레임은 지속적으로 완전히 갱신되어야 할 필요가 있죠. 이는 매우 비효율적이고 더 나아가선 모든 코스트를 제외할 필요를 야기할 수 있습니다.

    2020년의 MLX 버전 부터는 프레임을 synchronize 시키는 것이 가능해졌습니다. 이는 임시 방편으로 다수의 이미지를 스크린 버퍼링을 위해 만들 필요가 사라졌다는 것을 의미합니다.
2.  Using sync

    우선 이해해야 하는 걸 먼저 정의하고 시작하겠습니다.

    ```c
    #define MLX_SYNC_IMAGE_WRITABLE		1
    #define MLX_SYNC_WIN_FLUSH_CMD		2
    #define MLX_SYNC_WIN_CMD_COMPLETED	3

    int	mlx_sync(int cmd, void *ptr);
    ```

    `mlx_sync` 함수는 정의된 커맨드 코드들로 호출될 수 있습니다. 첫번째로 `MLX_SYNC_IMAGE_WRITABLE` 은 이미지에 대한 모든 다음 호출을 버퍼해줍니다. (`ptr` 은 MLX image object 의 포인터입니다.) 만약 변화를 증식시키길 원한다면, 당신은 이미지가 표시되는 중인 창을 `MLX_SYNC_WIN_FLUSH_CMD` 를 활용하고, `ptr` 에대해 flush를 원하는 창을 flush 할 필요가 있습니다.
3.  Test your skills!

    이전에 반복문을 활용해 만든 작은 circle-game 에 `mlx_syncd` 을 추가하여 렌더링을 해보세요.
