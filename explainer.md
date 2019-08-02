# Screen Enumeration / Window Placement

## Abstract

As it becomes more common to use more than one monitor, it becomes more important to give Web developers the tools to make their applications perform well across multiple displays.

The Window Placement API allows developers to configure the placement of one or more browser windows across one or more screens. Placement encompasses the position (x-, y-, z-coordinates) and size of the window, in addition to more complex behavior around dragging, aligning, and resizing windows.

The Screen Enumeration API gives developers access to a list of the available screens and the display properties of each screen. It provides the foundation for some Window Placement APIs and can be used to enhance others.

## Use cases

* **Slide show presentation using multiple screens**
  * Open the presentation, speaker notes, and presenter controls on different screens in fullscreen mode.
  * Swap the presentation and notes (i.e. change the screen on which each window appears).
  * Open the speaker notes on a specific screen, not in fullscreen mode.
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

## Goals

## Non-Goals

## Proposal

### Existing capabilities

## Privacy & Security
