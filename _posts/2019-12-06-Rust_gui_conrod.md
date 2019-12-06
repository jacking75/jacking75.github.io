---
layout: post
title: Rust - conrod을 이용한 GUI 프로그래밍
published: true
categories: [Rust]
tags: rust gui conrod
---
[출처](https://qiita.com/ogata-k/items/0a27e82ab3dc2636ea07 )  
  
[conrod](https://github.com/PistonDevelopers/conrod)는 [piston](https://www.piston.rs/)라는 게임 엔진을 개발 중인 곳에서 만들고 있는 Rust 정품의 GUI 라이브러리이다.  
widget 구성과 관리, widget에 대한 이벤트의 전파 등 기본적인 기능을 담당할 수 있다.  
그러나 실제의 렌더링이나 OS에서 행사의 수취는 conrod는 conrod에 마련된 렌더링 용의 [glium(OpenGL)](https://github.com/glium/glium), 이벤트 관리용의 [winit](https://github.com/tomaka/winit)의 백엔드가 준비되어 있어서 이 것들을 사용한다.  
conrod에서 만들어진 GUI는 멀티 플랫폼에서 동작하므로 쉽게 다른 OS에 대응할 수 있다.
  
conrod는 widget 마다 Id를 준비하고 관리한다. 복잡해질수록 Id 준비가 좀 어려워서 관리하기 어려워지지만 Id의 유일성에서 widget의 변수 이름 같은 취급을 할 수 있는 띠이다. 이것에 의해 취득한 Id로 widget를 판단하거나, widget를 지정하여 호출하는 것이 간단하다.  
  
구체적인 예가 [여기에 몇 가지](https://github.com/PistonDevelopers/conrod/tree/master/backends/conrod_glium/examples) 들어 있으므로 도움이 될 것 같다.  
  
  
## 코드
카운트 업을 하는 프로그램이다.  
main.rs  
```
#[macro_use]
extern crate conrod;
extern crate find_folder;

use conrod::{widget, color, Colorable, Borderable, Sizeable, Positionable, Labelable, Widget};
use conrod::backend::glium::glium;
use conrod::backend::glium::glium::{DisplayBuild, Surface};

// 사용하는 id 리스트
widget_ids!(
    struct Ids {
        canvas,
        num_lbl,
        button,
    });

fn main() {
  // 설정 값
    const TITLE: &'static str = "카운트 업";
    let width = 400;
    let height = 300;

    // window 만들기
    let display = glium::glutin::WindowBuilder::new()
        .with_dimensions(width, height)
        .with_title(TITLE)
        .build_glium()  // window 구축
        .unwrap();

    // Ui 작성
    let mut ui = conrod::UiBuilder::new([width as f64, height as f64]).build();

    // Ui에서 사용하는 Font를 assets 아래의 파일에서 font::Map에 추가
    let assets = find_folder::Search::KidsThenParents(3, 5)
        .for_folder("assets")
        .unwrap();
    let font_path = assets.join("fonts/NotoSans/NotoSans-Regular.ttf");
    ui.fonts.insert_from_file(font_path).unwrap();

    // id를 관리하기 위해 관리자 작성
    let ids = &mut Ids::new(ui.widget_id_generator());

    // glium으로 렌더링 하기위해 renderer 준비
    let mut renderer = conrod::backend::glium::Renderer::new(&display).unwrap();

    // The widget과 image를 연결 시켜서 관리하는 mapping
    let image_map = conrod::image::Map::<glium::texture::Texture2d>::new();


    let mut num = "0".to_string();

    let mut event_loop = EventLoop::new();
    // window 이벤트 루프
    'main: loop {
        for event in event_loop.next(&display) {
            // window 이벤트의 핸들러를 ui에 세트시킨다
            if let Some(event) = conrod::backend::winit::convert(event.clone(), &display) {
                ui.handle_event(event);
                event_loop.needs_update();
            }

            match event {
                // Escape는 window 삭제 용으로
                glium::glutin::Event::KeyboardInput(
                    _,
                    _,
                    Some(glium::glutin::VirtualKeyCode::Escape),
                ) |
                glium::glutin::Event::Closed => break 'main,
                _ => {}
            }
        }


        set_widgets(ui.set_widgets(), ids, &mut num);

        // Ui 렌더와 표시
        if let Some(primitives) = ui.draw_if_changed() {
            renderer.fill(&display, primitives, &image_map);
            let mut target = display.draw();
            target.clear_color(0.0, 0.0, 0.0, 1.0);
            renderer.draw(&display, &mut target, &image_map).unwrap();
            target.finish().unwrap();
        }
    }
}

// 배치하는 widget 배치 방법 지정 함수
fn set_widgets(ref mut ui: conrod::UiCell, ids: &mut Ids, num: &mut String) {
  // 배경(canvas)
    widget::Canvas::new()
        .pad(0.0)
        .color(conrod::color::rgb(0.2, 0.35, 0.45))
        .set(ids.canvas, ui);

    // canvas의 id를 사용하여  지정하는 것으로 ui에서 canvas의 종과 횡 배열을 얻는다
    let canvas_wh = ui.wh_of(ids.canvas).unwrap();


    // 수치 표시
    widget::Text::new(num)
        .middle_of(ids.canvas)
        .font_size(140)  // 폰트 사이즈 지정！！
        .color(color::WHITE)  // 색 지정도 간단하게 할 수 있다
        .set(ids.num_lbl, ui);

    // 카운트 버튼
    if widget::Button::new()
        .w_h(canvas_wh[0] - 10.0, 40.0)  // 횡
        .mid_bottom_with_margin_on(ids.canvas, 5.0)// 위치
        .rgb(0.4, 0.75, 0.6)  // 색
        .border(2.0)  // 경계
        .label("count +1")
        .set(ids.button, ui)
        .was_clicked()
    {  // if 식 실행 부분
        if let Ok(count) = num.parse::<u32>() {
            *num = (count+1).to_string();
        } else {
            println!("invalid number");
        }
    }

}

// 이벤트 관리용 구조체
struct EventLoop {
    ui_needs_update: bool,
    last_update: std::time::Instant,
}

impl EventLoop {
    pub fn new() -> Self {
        EventLoop {
            last_update: std::time::Instant::now(),
            ui_needs_update: true,
        }
    }

    /// 모든 갱신 대상이 되는 이벤트를 위한 순차 취득용 함수
    pub fn next(&mut self, display: &glium::Display) -> Vec<glium::glutin::Event> {
        // 60FPS 보다 빠르게 되지 않게 하기 위해 바로 앞의 갱신 대상에서 적어도 16ms 기다리도록 한다
        let last_update = self.last_update;
        let sixteen_ms = std::time::Duration::from_millis(16);
        let duration_since_last_update = std::time::Instant::now().duration_since(last_update);  // 전 회의 갱신 시와 지금 시간 차를 취득
        if duration_since_last_update < sixteen_ms {
            std::thread::sleep(sixteen_ms - duration_since_last_update);
        }

        // 이벤트 전체 취득
        let mut events = Vec::new();
        events.extend(display.poll_events());  // display에 있어서 이벤트 취득

        // display에서 갱신이 있으면 Ui에서 이벤트 갱신은 다음 대기를 넘는다
        if events.is_empty() && !self.ui_needs_update {
            events.extend(display.wait_events().next());
        }

        // 이벤트 갱신 후의 처리
        self.ui_needs_update = false;
        self.last_update = std::time::Instant::now();

        events
    }

    // Ui에서 다른 이벤트 갱신이 있는지 없는지를 요구하는 것을 event 루프에서는 확인해 둔다

    // 이것은 몇개의 Ui를 렌더링 하는 최초의 타이밍이나 갱신을 요구하는 타이밍에서 사용된다.
    pub fn needs_update(&mut self) {
        self.ui_needs_update = true;
    }
}
```
  
  
  
## 코드 설명
우선 이번 프로그램에서 사용하는 Id를 관리하기 위한 구조체를 매크로로 생성한다.  
```
widget_ids!(
    struct Ids {
        canvas,
        num_lbl,
        button,
    });
```
  
그리고 window를 만든다.
다음으로 레이아웃을 관리하는 ui를 만들고, 필요한 assets(이번에는 폰트)을 ui에 추가하고 있다.  
그리고. ui용의 Id를 생성한다.  
그리고 다음으로 glium으로 렌더링 하기 위한 renderer 준비를 하고, 이 때 사용하는 widget과 이 widget 용의 이미지를 대응시키는 사전을 만든다.  
마지막으로 event 주위, 데이터 변수와 ui와 Id를 연결 시키고, 다시 렌더 관련 관계를 준비하고 main 함수는 종료한다.  
  
  
