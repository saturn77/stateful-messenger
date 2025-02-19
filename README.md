# stateful-messenger
A library for message passing between upstream logic and an Eframe instance, facilitating isolation of business logic and cleaner code.

## Software Requirements 

The requirements frame and focus the scope of the library, and are helpful guideposts when ascertaining which features to put into the software. 

- Use of traits to define generic featurs in all the messages
- Asynchronous by nature
- Handshaking from the upstream process and the eframe instance
- State machine to handle the process of transferring a message
- Readily modified Rust code; may employ advanced features but emphasis on making the code maintainable and readable

## Introduction

The Egui library is an effective user interface that is based on immediate mode principles, and there is often 
a mixture of business logic and presentation logic in Egui applications. The idea of a formal message library 
between an upstream process and an efram instance is designed to increase the readability, usability, and maintenance
of Egui projects. 

The principle idea is based on 

```rust
eframe::run_native(
        gui::WINDOW_TITLE,
        options,
        Box::new(move |cc| {
            egui_extras::install_image_loaders(&cc.egui_ctx);
            Ok(Box::new(MyApp::new(cc, shared_state.clone())))
        }),
    )
```

where shared_state can be modified by the logic called in main.rs or in the eframe instance itself. 


## Flexibility

With the above in mind, how to make the library flexible ? 

A basic example of this is shown below : 

```rust
pub trait GenericMessageTrait {
    type GenericMessageContentType;
    fn set_message(&mut self, message: Self::GenericMessageContentType);
    fn next_state_logic(&mut self);
}

enum MessageState {
    Idle, 
    NewMessageDetected,
    ConfirmMessage,
}

struct MessageTypeUsize {
    state               : MessageState,
    message             : usize, 
    previous            : usize, 
    ready_from_upstream : bool,
    valid_from_gui      : bool,
}

impl GenericMessageTrait for MessageTypeUsize {

    type GenericMessageContentType = usize;

    // Set the message from the upstream process 
    // (main.rs or main.rs calling a given business 
    // logic function), and  in this case the message 
    // is a usize. For a 64-bit system, this is a 64-bit 
    // unsigned integer. For a 32-bit system, this is 
    // a 32-bit unsigned integer.
    fn set_message(&mut self, message: usize) {
        self.message = message;
        self.ready_from_upstream = true;
    }

    // make a simple state machine that will look for a new message
    // and then confirm the message.
    fn next_state_logic(&mut self) {

        match self.state {
            MessageState::Idle => {
                if (self.message != self.previous) && !self.valid_from_gui {
                    self.state = MessageState::NewMessageDetected;
                } else {
                    self.state = MessageState::Idle;
                }
            },
            MessageState::NewMessageDetected => {
                if self.valid_from_gui {
                    self.state = MessageState::ConfirmMessage;
                } else {
                    self.state = MessageState::NewMessageDetected;
                }
            },
            MessageState::ConfirmMessage => {
                self.valid_from_gui = false;
                self.state = MessageState::Idle;
            },
        
        }
    }

}
```



