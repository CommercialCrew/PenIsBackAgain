# Win2D를 이용한 그림판 만들기

## 실행화면
![캡처](https://github.com/CommercialCrew/PenIsBackAgain/assets/101386134/1c3c2e51-ff71-40d4-828e-32d2a8224343)

## 제작과정

1. 프로젝트 - NuGet 패키지 관리에 들어가서 Microsoft.Graphics.Win2D 패키지의 1.0.5.1 버전을 설치한다. Microsoft에서 직접 제공하는 Win2D 패키지다.
왜 굳이 1.0.5.1 버전을 이용하냐면 1.1.0 버전 부터 있던 이슈인데, 프로젝트 작성 후 실행을 하는데 있어서 문제가 있다. 현 시점 최신버전인 1.1.1 버전을 이용해도 똑같다.

2. 기본으로 버튼이 추가되어 있는데, Xaml, Xaml.cpp, Xaml.h 파일에서 버튼 관련 요소들을 모두 지운 뒤 프로젝트를 한번 실행해본다. 정상작동 여부는 한번 확인해볼 필요가 있다.

3. Xaml 파일을 먼저 작성한다.
<Window ~~~ /> 부분에
xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
를 추가한다. Win2D 패키지에서 추가된 것으로, Win2D의 Canvas 요소를 이용하려면 사용해야 한다.

그 다음에는 다음 코드를 추가한다.

<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="60" />
        <RowDefinition Height="590" />
    </Grid.RowDefinitions>

    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="600" />
        <ColumnDefinition Width="400" />
    </Grid.ColumnDefinitions>

        <Slider AutomationProperties.Name="color slider" Width="200"
                Grid.Column="0" Grid.Row="0"
                ValueChanged="Slider_ValueChanged" />

        <canvas:CanvasControl Grid.Column="0" Grid.Row="1"
                              PointerPressed="CanvasControl_PointerPressed"
                              PointerReleased="CanvasControl_PointerReleased"
                              PointerMoved="CanvasControl_PointerMoved"
                              Draw="CanvasControl_Draw"
                              ClearColor="Aquamarine" />

        <AppBarToggleButton Grid.Column="1" Grid.Row="0"
                            x:Name="ColorPicker" Icon="Shuffle" IsChecked="True"
                            Label="Enable" Click="ColorPicker_Click" />

        <Border Grid.Column="1" Grid.Row="1" x:Name="colorPanel" Visibility="Visible">
            <ColorPicker
                ColorChanged="ColorPicker_ColorChanged"
                ColorSpectrumShape="Ring"
                IsMoreButtonVisible="False"
                IsColorSliderVisible="True"
                IsColorChannelTextInputVisible="True"
                IsHexInputVisible="True"
                IsAlphaEnabled="False"
                IsAlphaSliderVisible="True"
                IsAlphaTextInputVisible="True" />
        </Border>
</Grid>

Grid를 이용해서 영역을 나누는데, 그 크기를 RowDefinition과 ColumnDefinition으로 미리 정해두는 방식이다. 이후 Grid.Row=""과 Grid.Column=""을 이용해 Definition을 끌고올 수 있다.

이후에는 슬라이더, 그림을 그릴 캔버스, 색상표 표시여부를 결정하게 해주는 토글 버튼, 그리고 색상표 요소를 추가해준다.

4. 헤더 파일을 작성한다.

```#include<winrt/Microsoft.Graphics.Canvas.UI.Xaml.h>
#include<winrt/Microsoft.UI.Xaml.Input.h>
#include<winrt/Microsoft.UI.Input.h>

using namespace winrt::Microsoft::UI;
using namespace winrt::Microsoft::Graphics::Canvas::UI::Xaml;
using namespace std;
```

Win2D 프로그래밍에 필요한 요소들을 불러온다.

```namespace winrt::PenIsBackAgain::implementation
{
    struct MainWindow : MainWindowT<MainWindow>
    {
        MainWindow()
        {
            
            InitializeComponent();
        }

        int32_t MyProperty();
        void MyProperty(int32_t value);
        bool flag;
        bool IsChecked;
        float px;
        float py;
        float mySize;
        vector<float> vx;
        vector<float> vy;
        vector<struct winrt::Windows::UI::Color> col;
        vector<float> sizeP;
        void Slider_ValueChanged(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Controls::Primitives::RangeBaseValueChangedEventArgs const& e);
        void ColorPicker_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void ColorPicker_ColorChanged(winrt::Microsoft::UI::Xaml::Controls::ColorPicker const& sender, winrt::Microsoft::UI::Xaml::Controls::ColorChangedEventArgs const& args);
        void CanvasControl_PointerPressed(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void CanvasControl_PointerReleased(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void CanvasControl_PointerMoved(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void CanvasControl_Draw(winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasControl const& sender, winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasDrawEventArgs const& args);
    };
}
```

이후에는 이 부분을 작성한다. 뒷부분의 void 함수들은 Xaml파일에서 처리기 추가 버튼을 눌러 추가했으면 자동생성되는 부분이고, 우리는 중간의
int32_t MyProperty(); 부분 바로 아래의 bool과 float, vector 변수들만 작성하면 된다.

5. 마지막으로 cpp 파일을 작성한다.

```struct winrt::Windows::UI::Color ColorP = winrt::Microsoft::UI::Colors::Green();```

으로 색상을 미리 변수에 저장해두고,

```
namespace winrt::PenIsBackAgain::implementation
{

    

    int32_t MainWindow::MyProperty()
    {
        flag = false;
        px = 100;
        py = 100;
        mySize = 16;
        IsChecked = true;
        throw hresult_not_implemented();
    }

    void MainWindow::MyProperty(int32_t /* value */)
    {
        throw hresult_not_implemented();
    }
    
}
```
헤더파일에서 작성한 변수는 이렇게 초기화 해준다. 변수 초기화 부분 말고는 전부 이미 작성되어 있는 부분이다.
(책에서 이 부분에 실수가 있었던 것으로 보인다. 버전이슈인지는 모르겠지만 현재는 이렇게 넣어야 정상적으로 헤더에서 선언한 변수를 인식하고 초기화가 가능하다.)



```
void winrt::PenIsBackAgain::implementation::MainWindow::Slider_ValueChanged(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Controls::Primitives::RangeBaseValueChangedEventArgs const& e)
{
    mySize = e.NewValue();
}


void winrt::PenIsBackAgain::implementation::MainWindow::ColorPicker_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    if (IsChecked) {
        IsChecked = FALSE;
        ColorPicker().Label(L"Disable");

        colorPanel().Visibility(Visibility::Collapsed);
    }
    else {
        IsChecked = TRUE;
        ColorPicker().Label(L"Enable");

        colorPanel().Visibility(Visibility::Visible);
    }
}

void winrt::PenIsBackAgain::implementation::MainWindow::ColorPicker_ColorChanged(winrt::Microsoft::UI::Xaml::Controls::ColorPicker const& sender, winrt::Microsoft::UI::Xaml::Controls::ColorChangedEventArgs const& args)
{
    ColorP = args.NewColor();
}


void winrt::PenIsBackAgain::implementation::MainWindow::CanvasControl_PointerPressed(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    flag = true;
    
}


void winrt::PenIsBackAgain::implementation::MainWindow::CanvasControl_PointerReleased(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    flag = false;
    px = py = -0.0;
    vx.push_back(px);
    vy.push_back(py);
    col.push_back(ColorP);
    sizeP.push_back(mySize);
}


void winrt::PenIsBackAgain::implementation::MainWindow::CanvasControl_PointerMoved(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    CanvasControl canvas = sender.as<CanvasControl>();
    px = e.GetCurrentPoint(canvas).Position().X;
    py = e.GetCurrentPoint(canvas).Position().Y;
    if (flag) {
        vx.push_back(px);
        vy.push_back(py);
        col.push_back(ColorP);
        sizeP.push_back(mySize);
        canvas.Invalidate();
    }
}


void winrt::PenIsBackAgain::implementation::MainWindow::CanvasControl_Draw(winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasControl const& sender, winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasDrawEventArgs const& args)
{
    CanvasControl canvas = sender.as<CanvasControl>();
    int n = vx.size();
    if (n <= 2) return;

    for (int i = 1; i < n; i++) {
        if (vx[i] == 0.0 && vy[i] == 0.0) {
            i++;
            continue;
        }

        args.DrawingSession().DrawLine(vx[i - 1],vy[i - 1], vx[i], vy[i], col[i], sizeP[i]);
        args.DrawingSession().FillCircle(vx[i - 1], vy[i - 1], sizeP[i] / 2, col[i]);
        args.DrawingSession().FillCircle(vx[i], vy[i], sizeP[i] / 2, col[i]);
    }
}
```

이후는 모두 처리기 내용 작성 부분이다. 모든 함수들은 모두 처리기 작성 버튼을 눌러 생성시 자동으로 작성되어 있으며, 우리는 그 안의 코드만 입력하면 된다.
마우스의 위치 좌표를 px, py에 저장해두고 그 위치에 따라 선을 작성하는 식으로 작동되는데, 그냥 float변수인 px,py를 이용하지 못하므로 vector 변수인 vx,vy에 push_back()을 이용해 값을 물려주어 이용한다.
선을 그릴지의 여부는 bool flag를 이용해 판단한다.

그 외에 ColorPicker 처리기도 정의하여 색 변환 부분을 구현한다.

