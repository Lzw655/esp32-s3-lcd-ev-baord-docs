.. ================
.. Revision History
.. ================
.. - v1.0, April 2022
..   * First release

.. - This is the user guide template for functional dev boards. 
.. - For the instructions on this template, drawing templates, raw material checklist, or other useful documents when preparing user guides, please go to the One-Stop Page for Preparing Dev Board User Guide (referred to as "One-Stop Page" below) at https://espressifsystems.sharepoint.com/sites/Documentation/SitePages/Dev-Board-User-Guide.aspx


===================
[Name of Board]
===================

:link_to_translation:`zh_CN:[中文]`

.. - Semi-fixed content - Introductory words about this user guide.
.. - Provider - Doc Team.
This user guide will help you get started with [Name of Board] and will also provide more in-depth information.

.. - Fixed content - Description of the structure of the user guide.
The document consists of the following sections:

- `Board Overview`_: Overview of the board hardware/software.
- `Start Application Development`_: How to set up hardware/software to develop applications.
- `Hardware Reference`_: More detailed information about the board's hardware.
- `Hardware Revision Details`_: Hardware revision history, known issues, and links to user guides for previous versions (if any) of the board.
- `Ordering`_: How to buy the board.
- `Related Documents`_: Links to related documentation.


Board Overview
==============

.. - Varying content - Brief description of the board, such as chip integrated, development framework, application scenarios, and key features and functionalities that this board can demonstrate yet others cannot.
.. - Provider - Software Team.
.. - Example-starts:

The ESP32-S3-EYE is a small-sized AI development board produced by Espressif. It is based on the ESP32-S3 SoC and ESP-WHO, Espressif's AI development framework. With its display and audio features, it is perfect for image recognition and audio processing. ESP32-S3-EYE offers plenty of storage, with an 8 MB Octal PSRAM and an 8 MB flash. It also supports image transmission via Wi-Fi and debugging through a Micro-USB port. With ESP-WHO, you can develop a variety of AIoT applications, such as smart doorbells, surveillance systems, facial recognition time clocks, etc.

.. - Example-ends.

.. - Semi-fixed content - Directive to include isometric view (three-dimensional) photo of the board.
.. - Provider - Doc Team.
.. - Note 
..   * The Directive in this content block refers to the reStructureText (reST) syntax below to include the actual isometric view of the board. It starts with ".. figure:: ../../../_static/???.???" and ends with "[Name of Board] with [Name of Module] module". For which team should prepare the actual photo, please refer to the Raw Material Checklist at the One-Stop Page.
..   * The path to figures varies from repo to repo.
.. figure:: ../../../_static/???.???
    :align: center
    :alt: [Name of Board] with [Name of Module] module
    :figclass: align-center

    [Name of Board] with [Name of Module] module


Feature List
------------

.. - Fixed content.
The main features of the board are listed below:

.. - Semi-fixed - List of board features.
.. - Provider - Hardware Team.
.. - Note - The "[Brief description of feature]" could be omitted for self-explanatory features. 
- **[Feature Name]:** [Brief description of feature]
- ...
.. - Example-starts:

- **Module Embedded:** ESP32-S2-WROVER module with 4 MB flash and 2 MB PSRAM
- **Display:** 4.3-inch TFT-LCD which uses 16-bit 8080 parallel port with 480×800 resolution and 256-level hardware DC backlight adjustment circuit, connected to an I2C capacitive touch panel
- **Audio:** Audio amplifier, built-in microphone, speaker connector
- **Storage:** microSD card connector
- **Sensors:** 3-axis accelerometer, 3-axis gyroscope, ambient light sensor, temperature and humidity sensors
- **Expansion:** SPI header, TWAI interface (compatible with CAN 2.0), I2C ADC, UART/Prog header
- **LEDs:** Programmable RGB LED and IR LED
- **Buttons:** Wake Up and Reset buttons
- **USB:** 1 x USB-C OTG (DFU/CDC) port, 1 x USB-C debug port
- **Power Supply:** 5V and 3.3V power headers
- **Optional Rechargeable Battery:** 1,950 mAh single-core lithium battery with a charge IC

.. - Example-ends.


Block Diagram
-------------

.. - Semi-fixed content - Introduction to the block diagram.
.. - Provider - Doc Team.
The block diagram below shows the components of [Name of Board] and their interconnections.

.. - Semi-fixed content - Directive to include the block diagram showing main components and their interconnections.
.. - Provider - Doc Team.
.. - Note - The path to figures varies from repo to repo.
.. figure:: ../../../_static/???.???
    :align: center
    :scale: 70%
    :alt: [Name of Board] (click to enlarge)
    :figclass: align-center

    [Name of Board] (click to enlarge)


Description of Components
-------------------------

.. - Semi-fixed content - Directive to include the picture of the board with annotations for an in-depth overview of components.
.. - Provider - Doc Team.
.. - Note:
..   * The path to figures varies from repo to repo.
..   * If more than one picture showing the board/kit from different angles is required, add "- front" for front view, "- back" for back view, etc.

.. figure:: ../../../_static/???.???
    :align: center
    :alt: [Name of Board] - front
    :figclass: align-center

    [Name of Board] - front

.. - Semi-fixed content - Introduction to the Description of Components table.
.. - Provider - Hardware Team.
The key components of the board are described in a [clockwise or counter-clockwise] direction.

.. - Semi-fixed content - Description of called-out components. 
.. - Provider - Hardware Team.
.. - Note:cd
..   * Fit the component names and descriptions in the following directive.
..   * For recommended descriptions, please refer to Recommended Description of Components at the One-Stop Page.
.. list-table::
   :widths: 30 70
   :header-rows: 1

   * - Key Component
     - Description
   * - [Component Name 1]
     - [Component description 1]
   * - [Component Name 2]
     - [Component description 2]
   * - ...
     - ...
.. - Example-starts:

.. list-table::
   :widths: 30 70
   :header-rows: 1

   * - Key Component
     - Description
   * - Function Press Keys
     - Six press keys labeled REC, MUTE, PLAY, SET, VOL- and VOL+. They are routed to ESP32-S3-WROOM-1 module and intended for development and testing of a UI for audio applications using dedicated API.
   * - Boot/Reset Press Keys
     - Boot: holding down the Boot button and momentarily pressing the Reset button initiates the firmware upload mode. Then you can upload firmware through the serial port. Reset: pressing this button alone resets the system.
   * - Battery Charger
     - Constant current and constant voltage linear charger for single cell lithium-ion batteries AP5056. Used for charging of a battery connected to the Battery Socket over the Micro USB Port.
   * - Audio PA Chip
     - NS4150 is an EMI, 3 W mono Class D audio power amplifier, amplifying audio signals from audio codec chips to drive speakers.
   * - ...
     - ...

.. - Example-ends.


Default Firmware and Function Test
----------------------------------

.. - Semi-fixed content - Introductory to the "Default Firmware and Function Test" subsection.
.. - Provider - Software Team.
Each [Name of Board] comes with a pre-built default firmware that allows you to test its functions including [name of function 1] ... [name of function X]. This section describes how to test [name of function X] with the pre-built firmware.

.. - Semi-fixed content - List of required hardware for default firmware functional test.
.. - Provider - Software Team.
Firstly, prepare the hardware:

- [Name of Board]
- USB 2.0 cable (Standard-A to Micro-B), for USB power supply

Secondly, connect your hardware.

- Before powering up your board, please make sure that it is in good condition with no obvious signs of damage.

.. - Varying content - Hardware connection instructions for default firmware functional test.
.. - Provider - Software Team.
.. - Note - It should describe how users can check that the hardware connection has been done properly by describing e.g. pattern changes of status LEDs.
.. - Example-starts:

- Connect the board to a power supply through the **USB Port** using a USB cable. After the board is powered up, you will notice the following responses:
    - The **Module Power LED** turns on for a few seconds, indicating that the default firmware is being loaded.
    - The **Module Power LED** turns off, indicating the default firmware has been loaded. The board enters human face recognition mode by default.
    - The LCD display shows live video streaming.

.. - Example-ends.

.. - Semi-fixed content - Name of the function tested.
.. - Provider - Software Team.
Thirdly, start testing the [name of function X].

.. - Varying content - Detailed steps to test the function.
.. - Provider - Software Team.
.. - Example-starts:

- Toggle the **Power Switch** to ON. The red **5 V Power On LED** should turn on.
- Press the **Reset Button** on the main board.
- Activate the board with the default Chinese wake word “Hi 乐鑫” (meaning "Hi Espressif"). When the wake word is detected, the 12 RGB LEDs on the extension board glow white one by one, indicating that the board is waiting for a speech command.
- Say a command to control your board. The table below provides a list of default Chinese speech commands.

.. - Example-ends.


Software Support
----------------

.. - Semi-fixed content - Software development framework introduction and versions.
.. - Provider - Software Team.
.. - Note - If there is no ready document describing the versions supported for the board, delete the sentence "Find out the supported version of [Framework name] for the board is at [link to document]." 
[Framework name and link] is the development framework for [Name of Board]. Find out the supported version of [Framework name] for the board is at [link to document].

.. - Example-starts:

`ESP-ADF <https://github.com/espressif/esp-adf>`_ is the development framework for ESP32-S3-Korvo-2.

.. - Example-ends.

.. - Semi-fixed content - List of other software repositories that can help users to experiment with the functions of the board.
.. - Provider - Software Team.
Below are other software repositories developed by Espressif that may help you experiment with the functions of [Name of Board].

- [Repository name and link]: [Description of repository]
- ...
.. - Example-starts:

Below are other software repositories provided by Espressif that may help you experiment with the functions of ESP32-S3-Korvo-2.

- `esp32-camera <https://github.com/espressif/esp32-camera>`_: Drivers for image sensors.
- `esp-sr <https://github.com/espressif/esp-sr>`_: Algorithms for speech recognition applications.

.. - Example-ends.

.. - Semi-fixed content - Link to application examples.
.. - Provider - Software Team.
Application examples for this board can be found at [link to examples].

.. - Example-starts:

Application examples for this board can be found at :adf:`application example <examples>`.

.. - Example-ends.


Start Application Development
=============================

.. - Fixed content - Introductory words about this section.
This section provides instructions on how to do hardware and software setup and flash firmware onto the board to develop your own application.


Required Hardware
-----------------

.. - Semi-fixed content - List of required hardware for application development.
.. - Provider - Software Team/Hardware Team.
.. - Note:
..  * Software Team should provide the number and names of required hardware; 
..  * If "[Note or recommended specification for the hardware]" is needed, Hardware Team should provide it, either in the following list or organize the required hardware into a table and fill it in the column "Note" like the example below. 
- [number of hardware] x [Name of Board]
- [number of hardware] x USB 2.0 cable (Standard-A to Micro-B)
- [number of hardware] x Computer running Windows, Linux, or macOS
- [number of hardware] x [Name of hardware]
  
  [Note or recommended specification for the hardware]
- ...
- ...  
.. - Example-starts:

.. list-table::
   :widths: 30 10 70
   :header-rows: 1

   * - Hardware
     - QTY
     - Note
   * - ESP32-S3-Korvo-1
     - 1
     - –
   * - USB 2.0 cables (Standard-A to Micro-B)
     - 2
     - One for USB power supply, the other for flashing firmware onto the board. Be sure to use an appropriate USB cable. Some cables are for charging only and do not provide the needed data lines nor work for programming the boards.
   * - Computer running Windows, Linux, or macOS
     - 1
     - –
   * - Speaker or headphones
     - 1
     - The 4-ohm 3-watt speaker is recommended. It should be fitted with JST PH 2.0 2-Pin plugs. In case you do not have this type of plug it is also fine to use Dupont female jumper wires during development. Headphones with a 3.5 mm jack are recommended.

.. - Example-ends.

.. - Semi-fixed content - List of optional hardware for application development.
.. - Provider - Software Team/Hardware Team.
.. - Note:
..   * This content is optional depending on whether there is any optional hardware.
..   * Software Team should provide the number and names of optional hardware; Hardware Team should provide the "[Note or recommended specification for the hardware]" if they think the additional information is needed by customers to prepare this hardware.
- [number of hardware] x [Name of hardware]
  
  [Note or recommended specification for the hardware]
- [number of hardware] x [Name of hardware]
  
  [Note or recommended specification for the hardware]
.. - Example-starts:

Optional Hardware
-----------------
- 1 x MicroSD card  
- 1 x Li-ion battery
  
  The battery is an alternative power supply to the USB Power Port. Make sure to use a Li-ion battery that has a protection circuit and fuse. The recommended specifications of the battery: capacity > 1000 mAh, output voltage 3.7 V, input voltage 4.2 V - 5 V. Please verify if polarity on the battery plug matches polarity of the socket as marked on the board's soldermask besides the socket.

.. - Example-ends.

Power Supply Options
--------------------

.. - Varying content - List of ways to power up the board.
.. - Provider - Software Team.
.. - Example-starts:

There are [number of ways] ways to provide power to the board:

- USB Power Port
- External battery via the 2-pin battery connector

.. - Example-ends.

Hardware Setup
--------------

.. - Fixed content.
Prepare the board for loading of the first sample application:

.. - Varying content - Hardware setup instructions.
.. - Provider - Software Team.
.. - Example-starts:

1. Connect the board to a power supply through the **USB Power Port** using a USB cable. The **Battery Green LED** should turn on. Assuming that a battery is not connected, the **Battery Red LED** will blink.
2. Toggle the **Power Switch** to **ON**. The red **5 V Power On LED** should turn on.
3. Connect the board to the computer through the **USB-to-UART Port** using a USB cable.
4. Connect a speaker to the **Speaker Output**, or connect headphones to the **Headphone Output**.

.. - Example-ends.

.. - Fixed content.
Now the board is ready for software setup.

Software Setup
--------------

.. - Varying content - Guide to software setup.
.. - Provider - Software Team.
.. - Note -  Choose one of the two ways depending on the actual situation:
..   * Option 1: If there is a ready guide for software setup, provide the link to the guide as follows.
After hardware setup, you can proceed to [link to software guide] to prepare development tools.

..   * Option 2: Otherwise, list the major setup steps in this section for now. Later, after Software Team prepare and publish the guide, switch to Option 1.
.. - Example-starts:

Below are some major software setup steps.

- Get `ESP-IDF <https://github.com/espressif/esp-who/#get-esp-idf>`_ which provides a common framework to develop applications for ESP32-S3 in C language.
- Get `ESP-WHO <https://github.com/espressif/esp-who/#get-esp-who>`_ which is an image processing platform that runs on ESP-IDF.
- Run `examples <https://github.com/espressif/esp-who/#run-examples>`_ that are provided by ESP-WHO.

.. - Example-ends.

.. Fix-content.
For more software information on developing applications, please go to `Software Support`_.

Hardware Reference
==================

.. Fixed content - Introductory sentence to this section
This section provides more detailed information about the board's hardware.

GPIO Allocation
---------------

.. - Semi-fixed content - Directive to include the table of module GPIO allocation.
.. - Provider - Doc Team.
.. - Note - Choose one of the two ways depending on the actual situation:
.. - Option 1: If the table has too many columns and is likely to spill over the border after being rendered to HTML, convert the Excel file prepared by Hardware Team to PDF, place it in the "_static" folder, and provide a link to the table in the following introductory sentence. Usually, tables with eight and more columns cannot fit into the HTML page.
The :download:`table <../../_static/name-of-board-gpio-allocation.pdf>` provides the allocation of GPIOs exposed on terminals of [Name of Module] module to control specific components or functions of the board.

.. - Example-starts:

The :download:`table <../../_static/esp32-s3-korvo-2-gpio-allocation.pdf>` provides the allocation of GPIOs exposed on terminals of ESP32-S3-WROOM-1 module to control specific components or functions of the board.

.. - Example-ends.

.. - Option 2: If it is a small table and does not spill over the page border after being rendered to HTML, provide it in the reST format.

The table below provides the allocation of GPIOs exposed on terminals of [Name of Module] module to control specific components or functions of the board.

.. list-table:: [Name of Embedded Module] GPIO Allocation
   :header-rows: 1
   :widths: 10 10 10 10 10 10 10

   * - Pin
     - Pin Name
     - [Component/Function Name]
     - [Component/Function Name]
     - [Component/Function Name]
     - [Component/Function Name]
     - [Component/Function Name]
   * - 
     - 
     - 
     - 
     - 
     - 
     -
   * -
     - 
     -   
     - 
     - 
     - 
     -

.. - Example-starts:

.. list-table:: ESP32-S3-WROOM-1 GPIO Allocation
   :header-rows: 1
   :widths: 10 10 10 10 10 10 10

   * - Pin [#one]_
     - Pin Name
     - ES8311
     - ES7210
     - Camera
     - LCD
     - Keys
   * - 3
     - EN
     - 
     - 
     - 
     - 
     - EN_KEY


.. [#one] Pin - ESP32-S3-WROOM-1 module pin number, GND and power supply pins are not listed.

.. - Example-ends.    


Power Distribution
------------------


Power Supply over USB and from Battery
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. - Varying content - Introductory paragraph to power supplies.
.. - Provider - Hardware Team.
.. - Example-starts:

There are two ways to power the development board: 5 V USB Power Port or 3.7 V optional battery. The optional battery is preferable for applications where a cleaner power supply is required.

.. - Example-ends.

.. - Semi-fixed content - Directive to include the screenshots of schematics of all power supplies.
.. - Provider - Doc Team.
.. - Note:
..   * The path to the figure varies from repo to repo.
..   * The number of screenshots is determined by the number of power supplies.
.. figure:: ../../../_static/???.???
    :align: center
    :scale: 70%
    :alt: [Figure Name] 
    :figclass: align-center

    [Figure Name]

.. - Example-starts:

.. figure:: ../../../_static/esp32-s3-korvo-2-v3.0-usb-ps.png
    :scale: 40%
    :align: center
    :alt: ESP32-S3-Korvo-2 V3.0 - Dedicated USB Power Supply Socket
    :figclass: align-center

    ESP32-S3-Korvo-2 V3.0 - Dedicated USB Power Supply Socket

.. figure:: ../../../_static/esp32-s3-korvo-2-v3.0-battery-ps.png
    :scale: 40%
    :align: center
    :alt: ESP32-S3-Korvo-2 V3.0 - Power Supply from a Battery
    :figclass: align-center

    ESP32-S3-Korvo-2 V3.0 - Power Supply from a Battery

.. - Example-ends.

Independent Audio and Digital Power Supply
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. - Varying content - Introductory paragraph to independent power supplies.
.. - Provider - Hardware Team.
.. - Note - The boards dedicated to processing video may have independent module and camera power supplies instead of audio and digital power supplies.
.. - Example-starts:

The board features independent power supplies for the audio components and ESP module. This should reduce noise in the audio signal from digital components and improve the overall performance of the components.

.. - Example-ends.

.. - Semi-fixed content - Directive to include the screenshots of schematics of independent power supplies.
.. - Provider - Doc Team.
.. - Note:
..   * The path to the figure varies from repo to repo.
..   * The number of screenshots is determined by the number of power supplies.
.. figure:: ../../../_static/???.???
    :align: center
    :scale: 70%
    :alt: [Figure Name]
    :figclass: align-center

    [Figure Name]
.. - Example-starts:

.. figure:: ../../../_static/esp32-s3-korvo-2-v3.0-digital-ps.png
    :scale: 40%
    :alt: ESP32-S3-Korvo-2 V3.0 - Digital Power Supply
    :figclass: align-center

    ESP32-S3-Korvo-2 V3.0 - Digital Power Supply

.. figure:: ../../../_static/esp32-s3-korvo-2-v3.0-audio-ps.png
    :scale: 40%
    :alt: ESP32-S3-Korvo-2 V3.0 - Audio Power Supply
    :figclass: align-center

    ESP32-S3-Korvo-2 V3.0 - Audio Power Supply

.. - Example-ends.


[Other Subsections]
-------------------

.. - Varying content - Other necessary subsections, which demonstrate the major functionality of the board, customers frequently ask about, or the Software/Hardware Team want to highlight.
.. - Provider - Hardware Team/Software Team/Doc Team.
.. - Note - The following subsections are listed for reference.
..   * Selecting of Audio Output.
..   * Hardware Setup Options.
..   * AEC Path.


Selection of Audio Output
-------------------------

.. - Varying content - Introductory paragraph to the selection of audio output.
.. - Provider - Hardware Team.
.. - Example-starts:

The board provides two mutually exclusive audio outputs:

- **Headphone output**: If headphones are plugged in, headphone output is enabled; speaker output is disabled.
- **Speaker output**: If headphones are not plugged in, speaker output is enabled; headphone output is disabled.

.. - Example-ends.


Hardware Setup Options
----------------------

.. - Varying content - Introduction to hardware setup options, such as using automatic upload, enabling JTAG, enabling microSD card in 1-wire mode and in 4-wire mode.
.. - Provider - Hardware Team.


AEC Path
--------

.. - Varying content - Introduction to AEC path.
.. - Provider - Hardware Team.
.. - Example-starts:

The acoustic echo cancellation (AEC) path provides reference signals for AEC algorithm.

ESP32-S3-Korvo-2 provides two compatible echo reference signal source designs. One is Codec (ES8311) DAC output (DAC_AOUTLP/DAC_AOUTLP), the other is PA (NS4150) output (PA_OUT+/PA_OUT+). The former is a default and the recommended selection. Resistors R132 and R140 shown in the figure below and marked NC (no component) should not be installed.

The echo reference signal is collected by ADC_MIC3P/ADC_MIC3N of ADC (ES7210) and then sent back to ESP32-S3 for AEC algorithm.

.. figure:: ../../../_static/esp32-s3-korvo-2-v3.0-aec-codec-o.png
    :scale: 60%
    :alt: ESP32-S3-Korvo-2 V3.0 - AEC Codec DAC Output
    :figclass: align-center

    ESP32-S3-Korvo-2 V3.0 - AEC Codec DAC Output

.. figure:: ../../../_static/esp32-s3-korvo-2-v3.0-aec-pa-o.png
    :scale: 30%
    :alt: ESP32-S3-Korvo-2 V3.0 - AEC PA Output
    :figclass: align-center

    ESP32-S3-Korvo-2 V3.0 - AEC PA Output

.. figure:: ../../../_static/esp32-s3-korvo-2-v3.0-aec-signal-collection.png
    :scale: 60%
    :alt: ESP32-S3-Korvo-2 V3.0 - AEC Reference Signal Collection
    :figclass: align-center

    ESP32-S3-Korvo-2 V3.0 - AEC Reference Signal Collection

.. - Example-ends.


Hardware Revision Details
=========================

.. - If the board has only one version, add the phrase: "No previous revisions." Otherwise, add the sections below.


Revision History
----------------

..  - Varying content - Board revision history.
..  - Provider - Hardware Team.
..  - Note:
..    * If the board has no revision history, remove this subsection. Then, add the phrase "This is the first revision of this board released."
..    * Otherwise, add information about updates/improvements in the new version, why they were introduced, and what they changed. Include the revision history of previous versions (if any) starting from the most recent.
.. - Example-starts:

Compared to ESP32-S3-Korvo-1 v4.0, ESP32-S3-Korvo-1 v5.0 has two changes in hardware: 1) marking on the main board，2) location of J1 on the main board. The changes are described in detail below:

- Marking on the (back of) main board is "ESP32-S3-Korvo-1 V5.0" for ESP32-S3-Korvo-1 v5.0 or "ESP32-S3-Korvo V4.0" for ESP32-S3-Korvo-1 v4.0.
- The J1 component on the ESP32-S3-Korvo-1 v5.0 main board is slightly moved to the right. This does not affect the performance of the board.

.. - Example-ends.


Known Issues
------------

.. - Varying content - Description of issues and how to avoid/fix them.
.. - Provider - Hardware Team/Software Team.
.. - Note - If the board has no known issues, remove this section. Otherwise, add the description of issues and how to avoid/fix them.
.. - Example-starts:

The component C15 may cause the following issues on ESP32-DevKitC V4 boards:

- Board may boot into Download mode
- If you output clock on GPIO0, C15 may impact the signal

In case these issues occur, please remove the component. The figure below shows C15 highlighted in yellow.

.. figure:: ../../../_static/???.???
    :align: center
    :alt: ESP32-DevKitC V4 - Location of C15
    :figclass: align-center

    ESP32-DevKitC V4 - Location of C15 (yellow)

.. - Example-ends.


User Guides for Previous Versions
---------------------------------

.. - Varying content - Links to all existing user guides.
.. - Provider - Doc Team.
.. - Note - If there are no previous versions of the user guide, remove this section. Otherwise, provide links to all existing guides.


Ordering
========

.. - Semi-fixed content - Directive to include the isometric view (three-dimensional) photo of the board in its package.
.. - Provider - Doc Team.
.. - Note - If the board/kit comes in individual packages, provide the following directive. Otherwise, remove it.
.. figure:: ../../../_static/???.???
    :align: center
    :alt: [Name of Board] - package
    :figclass: align-center

    [Name of Board] - package
.. - Example-starts:

.. figure:: https://dl.espressif.com/dl/schematics/pictures/esp32s2-kaluga-1-kit-v1.3-package-3d.png
    :align: center
    :alt: ESP32-S2-Kaluga-1 - package
    :figclass: align-center

    ESP32-S2-Kaluga-1 - package

.. - Example-ends.

.. - Fixed content.
If you order a few samples, each board comes in an individual package. Each package contains: 

.. - Semi-fixed content - List of components in packages, such as cables, jumpers, and other hardware parts.
.. - Provider - Doc Team.
.. - Note - Doc team fills in the information according to input by Business Administration Department.
- [number of component] x [Component name]
- ...

The components [are/are not] assembled by default.

.. - Example-starts:

- 1 x ESP32-S3-Korvo-1 main board
- 1 x ESP32-Korvo-Mic sub board
- 1 x FPC cable
- 8 x screws
- 4 x studs

The components are assembled by default.

.. - Example-ends.

.. - Fixed content.
For retail orders, please go to https://www.espressif.com/en/company/contact/buy-a-sample.

For wholesale orders, please go to https://www.espressif.com/en/contact-us/sales-questions.


Related Documents
=================

.. - Semi-fixed content - Directive to include related documents.
.. - Provider - Doc Team.

- Datasheet

  - `[Name of Chip] Datasheet <insert your link here>`_ (PDF)
  - `[Name of Module] Datasheet <insert your link here>`_ (PDF)

- Schematic

  - `[Name of Board] Schematic <insert your link here>`_ (PDF)

- PCB Layout

  - `[Name of Board] PCB Layout <insert your link here>`_ (PDF)

- Dimensions

  - `[Name of Board] Dimensions <insert your link here>`_ (PDF)
  - `[Name of Board] Dimensions Source File <insert your link here>`_ (DXF) - You can view it with `Autodesk Viewer <https://viewer.autodesk.com/>`_ online

.. - Semi-fixed content - Other related documents.
.. - Provider - Doc Team/Hardware Team/Software Team.
.. - Note - If there are documents that do not fall into the above categories, add the item "Other".
.. - Example-starts:

- Other

  - `JTAG Debugging <https://esp-idf.readthedocs.io/en/latest/api-guides/jtag-debugging/index.html>`_

.. - Example-ends.


.. - Fixed content
For further design documentation for the board, please contact us at `sales@espressif.com <sales@espressif.com>`_.