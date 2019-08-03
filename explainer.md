# Screen Enumeration / Window Placement

## Abstract

As it becomes more common to use more than one monitor, it becomes more important to give Web developers the tools to make their applications perform well across multiple displays.

The Window Placement API allows developers to configure the placement of one or more browser windows across one or more screens. Placement encompasses the position (x-, y-, z-coordinates) and size of the window, in addition to more complex behavior around dragging, aligning, and resizing windows.

The Screen Enumeration API gives developers access to a list of the available screens and the display properties of each screen. It provides the foundation for some Window Placement APIs and can be used to enhance others.

## Use cases

* **Slide show presentation using multiple screens**
  * Open the presentation, speaker notes, and presenter controls on different screens in fullscreen mode.
    ```js
    const screens = window.screens;
    
    // Option 1: Blow up multiple elements living in a single window.
    const presentation = document.querySelector("#presentation");
    const notes        = document.querySelector("#notes");
    const controls     = document.querySelector("#controls");
    
    presentation.requestFullscreen({ screen: screens[0] });
    notes.requestFullscreen({ screen: screens[1] });
    controls.requestFullscreen({ screen: screens[2] });
    
    // Option 2: Blow up multiple windows.
    window.open("/presentation", "presentation", "fullscreen", screens[0]);
    window.open("/notes", "notes", "fullscreen", screens[1]);
    window.open("/controls", "controls", "fullscreen", screens[2]);
    ```
  * Swap the presentation and notes (i.e. change the screen on which each window appears).
    ```js
    const presentationWindow = window.open("", "presentation");
    const notesWindow        = window.open("", "notes");
    presentationWindow.moveTo(notesWindow.screen);
    notesWindow.moveTo(presentationWindow.screen);
    
    // TODO: How would the size of the window be affected after the move?
    // TODO: Would window.arrange({ window: screen }) be better?
    ```
  * Move the speaker notes to a specific screen, not in fullscreen mode.
    ```js
    const screen = window.screens[0];
    const notesWindow = window.open("", "notes");
    notesWindow.moveTo(screen);
    
    // TODO: Find out if notesWindow.moveTo(screen.availLeft, screen.availTop) would suffice.
    ```
* **Professional image editing tools with floating palettes**
  * Always keep the palettes on top of the main editor.
  * Synchronously move the palettes when the main editor moves.
* **Finance applications with multiple dashboards**
  * Starting the app opens all the dashboards across multiple screens.
  * Starting the app restores all the dashboards' positions from the previous session.
  * Align dashboards relative to each other, or to the screen.
  * Snap dashboards into place when moved, according to a pre-defined configuration of window positions.
* **Small form-factor applications, e.g. calculator, mini music player**
  * Launch the app with specific (or bounded) dimensions.

## Goals / Non-goals

The initial implementation will address only the slide show use case. All other use cases are left to future iterations of the API, though the initial API should be designed to accommodate them with minimal modifications.

## Proposal

### Existing capabilities

## Privacy & Security
