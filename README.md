# imgui-winit-support

This crate provides a winit-based backend platform for [`imgui-rs`].

A backend platform handles window/input device events and manages their state.

# Using the library

There are five things you need to do to use this library correctly:

1. Initialize a [`WinitPlatform`] instance.
2. Attach it to a winit [`Window`] with [`WinitPlatform::attach_window`].
3. Pass events to the platform (every frame) with [`WinitPlatform::handle_event`].
4. Call the frame preparation callback [`WinitPlatform::prepare_frame`] (every frame)
5. Call the render preparation callback [`WinitPlatform::prepare_render`] (every frame)

## Complete example (without a renderer)

```no_run
use imgui::Context;
use imgui_winit_support::{HiDpiMode, WinitPlatform};
use std::time::Instant;
use winit::event::{Event, WindowEvent};
use winit::event_loop::{ControlFlow, EventLoop};
use winit::window::Window;

let mut event_loop = EventLoop::new().expect("Failed to create EventLoop");
let mut window = Window::new(&event_loop).unwrap();

let mut imgui = Context::create();
// configure imgui-rs Context if necessary

let mut platform = WinitPlatform::init(&mut imgui); // step 1
platform.attach_window(imgui.io_mut(), &window, HiDpiMode::Default); // step 2

let mut last_frame = Instant::now();
let mut run = true;
event_loop.run(move |event, window_target| {
    match event {
        Event::NewEvents(_) => {
            // other application-specific logic
            let now = Instant::now();
            imgui.io_mut().update_delta_time(now - last_frame);
            last_frame = now;
        },
        Event::AboutToWait => {
            // other application-specific logic
            platform.prepare_frame(imgui.io_mut(), &window) // step 4
                .expect("Failed to prepare frame");
            window.request_redraw();
        }
        Event::WindowEvent { event: WindowEvent::RedrawRequested, .. } => {
            let ui = imgui.frame();
            // application-specific rendering *under the UI*

            // construct the UI

            platform.prepare_render(&ui, &window); // step 5
            // render the UI with a renderer
            let draw_data = imgui.render();
            // renderer.render(..., draw_data).expect("UI rendering failed");

            // application-specific rendering *over the UI*
        },
        Event::WindowEvent { event: WindowEvent::CloseRequested, .. } => {
            window_target.exit();
        }
        // other application-specific event handling
        event => {
            platform.handle_event(imgui.io_mut(), &window, &event); // step 3
            // other application-specific event handling
        }
    }
}).expect("EventLoop error");
```

## License

Licensed under either of

- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or https://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or https://opensource.org/licenses/MIT)

at your option.