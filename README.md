// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © JayRogers
//
//@version=4

study(title = "Previous Period Levels - X Alerts", shorttitle = "PPL-X", overlay = true, format = format.price)

// Code adapted, and extended from:
//  - https://www.tradingview.com/pine-script-docs/en/v4/essential/Drawings.html
//    https://www.tradingview.com/pine-script-docs/en/v4/essential/Drawings.html#examples-of-classic-indicators

////////////////////////////////////////////////////////////////////////////////
//
// ====== ABOUT THIS INDICATOR
//
//  - A simple but highly customisable display of previous higher time-frame
//    OHLC values, drawn using line.new and label.new. Nothing fancy yet but...
//
//  - Customised resolution input which excludes time frames lower than 1 hour
//    while extending the common higher reference inputs to include:
//
//    • 6, and 12 Hour
//    • 5 Day
//    • 3, and 6 Month
//    • 1 Year
//
//  - Alert conditions using an adjustable SMA to help reduce false positive
//    spam.
//
//  - Full visual customisation options for (almost) every aspect, so it can be
//    tuned to suit most individual preferences.
//
//  - In line with the miriad visual customisation options is the ability to
//    change the display format of the Labels, to show more or less information
//    and, even disable them altogether.
//
// ====== REASON FOR STUDY
//
//  - To practice advanced user input option handling to allow for a full visual
//    customisation experience without stepping outside of, or interfering with,
//    the intended function of the indicator.
//
//  - Provide reasonably clear code commenting and structure in order to be
//    useful as a potential learning aid for others, and future reference for
//    myself.
//
// ====== DISCLAIMER
//
//    Any trade decisions you make are entirely your own responsibility.
//    I've made an effort to squash all the bugs, but you never know!
//
////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////
//                                                                            //
//                       ====== OPTION LIST VARS ======                       //
//                                                                            //
//    * Setting up option list variables outside of the actual input can      //
//      make them much easier to work with if any comparison checks are       //
//      required, and can help keep subsequent code clean and readable.       //
//                                                                            //
////////////////////////////////////////////////////////////////////////////////

// -- resolution options.
i_res0 = "1 Hour",      i_res1 = "2 Hour",      i_res2 = "3 Hour"
i_res3 = "4 Hour",      i_res4 = "6 Hour",      i_res5 = "12 Hour"
i_res6 = "1 Day",       i_res7 = "5 Day",       i_res8 = "1 Week"
i_res9 = "1 Month",     i_res10 = "3 Month",    i_res11 = "6 Month"
i_res12 = "1 Year"

// -- header label formatting options.
i_header0 = "DISABLED"
i_header1 = "1M C"
i_header2 = "1M • C"
i_header3 = "1M Close"
i_header4 = "1M • Close"
i_header5 = "1 Month C"
i_header6 = "1 Month • C"
i_header7 = "1 Month Close"
i_header8 = "1 Month • Close"

// -- price label formatting and decimal accuracy options.
i_price0 = "DISABLED",          i_accu0 = "#.#"
i_price1 = "##.## ~",           i_accu1 = "#.##"
i_price2 = "P ##.## ~",         i_accu2 = "#.###"
i_price3 = "P • ##.## ~",       i_accu3 = "#.####"
i_price4 = "Price ##.## ~",     i_accu4 = "#.#####"
i_price5 = "Price • ##.## ~"

// -- line style options.
i_line0 = "Solid", i_line1 = "Dotted", i_line2 = "Dashed"

// -- label size options.
i_size0 = "Auto",   i_size1 = "Tiny",   i_size2 = "Small"
i_size3 = "Normal", i_size4 = "Large",  i_size5 = "Huge"

////////////////////////////////////////////////////////////////////////////////
//                                                                            //
//                            ====== INPUTS ======                            //
//                                                                            //
//               * Using the new 'inline' and 'group' feature *               //
//                                                                            //
////////////////////////////////////////////////////////////////////////////////

// -- resolutions dropdown list.
INP_resolution = input( i_res9,            title = "Reference Resolution",
      options = [ i_res0, i_res1, i_res2, i_res3, i_res4, i_res5,
      i_res6, i_res7, i_res8, i_res9, i_res10, i_res11, i_res12
      ])

// -- alert sources dropdown list.
INP_showTracker         = input( true,      title = "Showing ? ",
      inline = "alerts",                    group = "Alerts » SMA Visibility and Settings"
      )
INP_alertSource         = input( open,      title = "",
      inline = "alerts",                    group = "Alerts » SMA Visibility and Settings",
      type = input.source
      )
INP_alertLen            = input( 3,         title = "",
      inline = "alerts",                    group = "Alerts » SMA Visibility and Settings",
      type = input.integer, minval = 1
      )


// -- line styles dropdown list.
INP_showOpenActive      = input( true,      title = "Showing ? ",
      inline = "openActive",                group = "Open » Line Settings",     type = input.bool
      )
INP_lineActiveStyleO    = input( i_line0,   title = "",
      inline = "openActive",                group = "Open » Line Settings",
      options = [ i_line0, i_line1, i_line2
      ])
INP_lineActiveWidthO    = input( 2,         title = "",
      inline = "openActive",                group = "Open » Line Settings",     type = input.integer,
      minval = 1, maxval = 10
      )
INP_lineActiveColourO   = input( #42a5f5,   title = "",
      inline = "openActive",                group = "Open » Line Settings",     type = input.color
      )

INP_showOpenPrior       = input( false,     title = "Show previous extension ?  ",
      inline = "openPrior",                 group = "Open » Line Settings",     type = input.bool
      )
INP_linePriorStyleO     = input( i_line1,   title = "",
      inline = "openPrior",                 group = "Open » Line Settings",
      options = [ i_line0, i_line1, i_line2
      ])
INP_linePriorColourO    = input( #42a5f588, title = "",
      inline = "openPrior",                 group = "Open » Line Settings",     type = input.color
      )


// -- line styles dropdown list.
INP_showHighActive      = input( true,      title = "Showing ? ",
      inline = "highActive",                group = "High » Line Settings",     type = input.bool
      )
INP_lineActiveStyleH    = input( i_line0,   title = "",
      inline = "highActive",                group = "High » Line Settings",
      options = [ i_line0, i_line1, i_line2
      ])
INP_lineActiveWidthH    = input( 2,         title = "",
      inline = "highActive",                group = "High » Line Settings",     type = input.integer,
      minval = 1, maxval = 10
      )
INP_lineActiveColourH   = input( #d32f2f,   title = "",
      inline = "highActive",                group = "High » Line Settings",     type = input.color
      )

INP_showHighPrior       = input( false,     title = "Show previous extension ?  ",
      inline = "highPrior",                 group = "High » Line Settings",     type = input.bool
      )
INP_linePriorStyleH     = input( i_line1,   title = "",
      inline = "highPrior",                 group = "High » Line Settings",
      options = [ i_line0, i_line1, i_line2
      ])
INP_linePriorColourH    = input( #d32f2f88, title = "",
      inline = "highPrior",                 group = "High » Line Settings",     type = input.color
      )


// -- line styles dropdown list.
INP_showLowActive       = input( true,      title = "Showing ? ",
      inline = "lowActive",                 group = "Low » Line Settings",      type = input.bool
      )
INP_lineActiveStyleL    = input( i_line0,   title = "",
      inline = "lowActive",                 group = "Low » Line Settings",
      options = [ i_line0, i_line1, i_line2
      ])
INP_lineActiveWidthL    = input( 2,         title = "",
      inline = "lowActive",                 group = "Low » Line Settings",      type = input.integer,
      minval = 1, maxval = 10
      )
INP_lineActiveColourL   = input( #388e3c,   title = "",
      inline = "lowActive",                 group = "Low » Line Settings",      type = input.color
      )

INP_showLowPrior        = input( false,     title = "Show previous extension ?  ",
      inline = "lowPrior",                  group = "Low » Line Settings",      type = input.bool
      )
INP_linePriorStyleL     = input( i_line1,   title = "",
      inline = "lowPrior",                  group = "Low » Line Settings",
      options = [ i_line0, i_line1, i_line2
      ])
INP_linePriorColourL    = input( #388e3c88, title = "",
      inline = "lowPrior",                  group = "Low » Line Settings",      type = input.color
      )


// -- line styles dropdown list.
INP_showCloseActive     = input( true,      title = "Showing ? ",
      inline = "closeActive",               group = "Close » Line Settings",    type = input.bool
      )
INP_lineActiveStyleC    = input( i_line0,   title = "",
      inline = "closeActive",               group = "Close » Line Settings",
      options = [ i_line0, i_line1, i_line2
      ])
INP_lineActiveWidthC    = input( 2,         title = "",
      inline = "closeActive",               group = "Close » Line Settings",    type = input.integer,
      minval = 1, maxval = 10
      )
INP_lineActiveColourC   = input( #fbc02d,   title = "",
      inline = "closeActive",               group = "Close » Line Settings",    type = input.color
      )

INP_showClosePrior      = input( false,     title = "Show previous extension ?  ",
      inline = "closePrior",                group = "Close » Line Settings",    type = input.bool
      )
INP_linePriorStyleC     = input( i_line1,   title = "",
      inline = "closePrior",                group = "Close » Line Settings",
      options = [ i_line0, i_line1, i_line2
      ])
INP_linePriorColourC    = input( #fbc02d88, title = "",
      inline = "closePrior",                group = "Close » Line Settings",    type = input.color
      )


// -- header label format dropdown selections.
INP_labelHeaderFormat   = input( i_header8, title = "Header Label Format",
      options = [ i_header0, i_header1, i_header2, i_header3,
      i_header4, i_header5, i_header6, i_header7, i_header8
      ])

// -- price label format dropdown selections.
INP_labelPriceFormat    = input(i_price5,   title = "Price Label Format",
      options = [ i_price0, i_price1, i_price2, i_price3, i_price4, i_price5
      ])

// -- price label decimal accuracy dropdown selections.
INP_labelPriceAccuracy  = input(i_accu4,    title = "Price Label Accuracy",
      options = [ i_accu0, i_accu1, i_accu2, i_accu3, i_accu4
      ])

// -- label linebreak '\n' offset
INP_labelLineOffset     = input( 2,         title = "Labels Line Offset",       type = input.integer,
      minval = -5, maxval = 5
      )

// -- label sizes dropdown list.
INP_labelSize           = input( i_size3,   title = "Label Size",
      options = [ i_size0, i_size1, i_size2, i_size3, i_size4, i_size5
      ])

////////////////////////////////////////////////////////////////////////////////
//                                                                            //
//                          ====== FUNCTIONS ======                           //
//                                                                            //
////////////////////////////////////////////////////////////////////////////////

f_getResolution() =>
    // () Description:
    //  - Resolver for custom resolution input selection, converts input to
    //    compatible return string for security, output is also used for less
    //    verbose label text options.
    // Dependencies:
    //  - i_res1, i_res2, i_res3, i_res4, i_res5, i_res6
    //  - i_res7, i_res8, i_res9, i_res10, i_res11, i_res12
    // Notes:
    //  - i_res0 excluded as it's a token placeholder for default "60".

    string _r       = INP_resolution    // a more ternary challenge friendly var
    string _default = "60"              // if i_res0 was input, or failure.

    // compare input to determine proper string return for security calls.
    _return = _r == i_res1 ? "120"  : _r == i_res2 ? "180"  : _r == i_res3 ? "240"  :
              _r == i_res4 ? "360"  : _r == i_res5 ? "720"  : _r == i_res6 ? "D"    :
              _r == i_res7 ? "5D"   : _r == i_res8 ? "W"    : _r == i_res9 ? "M"    :
              _r == i_res10 ? "3M"  : _r == i_res11 ? "6M"  : _r == i_res12 ? "12M" : _default

f_getLineStyle( _inputStyle ) =>
    //  string _inputStyle : style selection input
    // () resolver for custom line style input selection, returns a usable
    //    line style type.
    // Dependencies:
    //  - i_line1, i_line2
    // Notes:
    //  * i_line0 omitted as we default to 'line.style_solid' anyway

    // compare input to determine proper line style to return.
    _return = _inputStyle == i_line1 ? line.style_dotted :
              _inputStyle == i_line2 ? line.style_dashed : line.style_solid

f_getLabelSize() =>
    // () resolver for custom label size input selection, returns a usable
    //    label size string.
    // Dependencies:
    //  - i_size1, i_size2, i_size3, i_size4, i_size5
    // Notes:
    //  * i_size0 omitted as we default to 'size.auto' anyway

    var string _s = INP_labelSize       // a more ternary challenge friendly var

    // compare input to determine proper label size to return.
    _return = _s == i_size1 ? size.tiny     :
              _s == i_size2 ? size.small    :
              _s == i_size3 ? size.normal   :
              _s == i_size4 ? size.large    :
              _s == i_size5 ? size.huge     : size.auto


f_getLineStyleSet( _string ) =>
    //  string _string : short identifier string, expected requests:
    //  - "O" | "H" | "L" | "C"
    // () gathers grouped styling input information based on string input for
    //    tuple return.

    var string  _activeStyle    = na
    var int     _activeWidth    = na
    var color   _activeColour   = na

    var bool    _priorShow      = na
    var string  _priorStyle     = na
    var color   _priorColour    = na

    _activeStyle   := _string == "O" ? INP_lineActiveStyleO :
                      _string == "H" ? INP_lineActiveStyleH :
                      _string == "L" ? INP_lineActiveStyleL : INP_lineActiveStyleC // "C"

    _activeWidth   := _string == "O" ? INP_lineActiveWidthO :
                      _string == "H" ? INP_lineActiveWidthH :
                      _string == "L" ? INP_lineActiveWidthL : INP_lineActiveWidthC // "C"

    _activeColour  := _string == "O" ? INP_lineActiveColourO :
                      _string == "H" ? INP_lineActiveColourH :
                      _string == "L" ? INP_lineActiveColourL : INP_lineActiveColourC // "C"

    _priorShow     := _string == "O" ? INP_showOpenPrior :
                      _string == "H" ? INP_showHighPrior :
                      _string == "L" ? INP_showLowPrior  : INP_showClosePrior // "C"

    _priorStyle    := _string == "O" ? INP_linePriorStyleO :
                      _string == "H" ? INP_linePriorStyleH :
                      _string == "L" ? INP_linePriorStyleL : INP_linePriorStyleC // "C"

    _priorColour   := _string == "O" ? INP_linePriorColourO :
                      _string == "H" ? INP_linePriorColourH :
                      _string == "L" ? INP_linePriorColourL : INP_linePriorColourC // "C"

    [ _activeStyle, _activeWidth, _activeColour, _priorShow, _priorStyle, _priorColour ]

f_setNewlineOffset( _string, _offset ) =>
    //  string  _string : label string
    //  int     _offset : number of breaks to add
    // () prepends or appends N amount of '\n' breaks to a string

    // init empty string to hold the '\n' stack
    var string _newLineStack = ""

    // create '\n' stack for setting vertical text offset if we have one
    if _offset != 0

        // only run the loop if the stack is empty
        if _newLineStack == ""

            // need to start at 0 so that we can properly run + and -
            for i = 0 to _offset

                // so we need to skip the zero pass, else 1 would give '\n\n'
                if i != 0

                    _newLineStack := _newLineStack + "\n"

    // return adjusted string, or original based on operation
    _return = _offset > 0 ? _string + _newLineStack :
              _offset < 0 ? _newLineStack + _string : _string

f_getHeaderText( _verbose, _short ) =>
    //  string  _verbose    : long text for label
    //  string  _short      : short text for label
    // () text formatting for label format input selection, returns string
    // Dependencies:
    //  - INP_labelHeaderFormat, INP_resolution
    //  - i_header1, i_header2, i_header3, i_header4
    //  - i_header5, i_header6, i_header7, i_header8
    //  * i_header0 is omitted here, it is used for boolean check for drawing
    //    in f_renderLevel() - no drawing? then no need for the string.
    //  - f_getResolution()
    
    string _f = INP_labelHeaderFormat   // a more ternary challenge friendly var

    // choose dropdown input text, or use security() compatible shorthand?
    string _periodString    = _f == i_header5 ? INP_resolution :
                              _f == i_header6 ? INP_resolution :
                              _f == i_header7 ? INP_resolution :
                              _f == i_header8 ? INP_resolution : f_getResolution()

    // choose full source name, or shorthand?
    string _sourceString    = _f == i_header3 ? _verbose :
                              _f == i_header4 ? _verbose :
                              _f == i_header7 ? _verbose :
                              _f == i_header8 ? _verbose : _short

    // set up the punctuation
    string _punctuation     = _f == i_header1 ? " " :
                              _f == i_header3 ? " " :
                              _f == i_header5 ? " " :
                              _f == i_header7 ? " " : " • "

    // return our fully concatenated string
    _return = _periodString + _punctuation + _sourceString

f_getPriceText( _float ) =>
    //  float   _float : float value for label string
    // () text formatting for label format input selection, returns string
    // Dependencies:
    //  - INP_labelPriceFormat, INP_labelPriceAccuracy
    //  - i_price1, i_price2, i_price3, i_price4, i_price5
    //  - i_accu0, i_accu1, i_accu2, i_accu3, i_accu4
    //  * i_price0 is omitted here, it is used for boolean check for drawing
    //    in f_renderLevel() - no drawing? then no need for the string.

    string _f = INP_labelPriceFormat    // a more ternary challenge friendly var

    // shorthand or verbose text? ..or just empty?
    string _heading     = _f == i_price2 ? "P" :
                          _f == i_price3 ? "P" :
                          _f == i_price4 ? "Price" :
                          _f == i_price5 ? "Price" : ""

    // set up the punctuation
    string _punctuation = _f == i_price1 ? " " :
                          _f == i_price2 ? " " :
                          _f == i_price4 ? " " : " • "

    string _price       = tostring( _float, INP_labelPriceAccuracy )

    // return our fully concatenated string
    _return = _heading + _punctuation + _price

f_renderLevel( _showing, _series, _verbose, _shorthand ) =>
    //  bool    _showing    : whether or not the level is to be drawn
    //  float   _series     : the price at ahich to draw the level
    //  string  _verbose    : long string for use in f_getHeaderText()
    //  string  _shorthand  : short string for use in f_getHeaderText()
    // () text formatting for label format input selection, returns string
    // Dependencies:
    //  - f_getResolution()
    //  - f_getLineStyle(), f_getLabelSize()
    //  - f_getHeaderText(), f_getPriceText(), f_setNewlineOffset()
    //  - i_header0, i_price0
    //  - INP_lineColour, INP_lineWidth
    //  - INP_labelHeaderFormat, INP_labelPriceFormat, INP_labelLineOffset

    // get a time() compatible resolution string from input
    string      _resolution     = f_getResolution()

    // set up line vars to hold active, and previous level
    var line    _lineActive     = na
    var line    _linePrior      = na

    // set up label vars for active, and previous level
    var label   _labelHeading   = na
    var label   _labelPrice     = na

    [ _activeStyle,
      _activeWidth,
      _activeColour,
      _priorShow,
      _priorStyle,
      _priorColour ] = f_getLineStyleSet( _shorthand )

    // if the resolution reference time changes
    if change( time( _resolution ) ) != 0

        // set up our x-location variables
        string  _xloc   = xloc.bar_time
        int     _x1     = time
        int     _x2     = time_close( _resolution )

        // if we are showing this level
        if _showing

            // LINE DRAWING

            // if previous line exists, delete it before moving on
            if not na( _linePrior )

                line.delete( _linePrior )

            // if we have an active, current line
            if not na( _lineActive )

                // if we are maintaining prior level
                if _priorShow

                    // transfer active line pointer to prior, and clear active
                    _linePrior := _lineActive, _lineActive := na

                    // update x2 to new end-point, and re-style
                    line.set_x2(    _linePrior, _x2 )
                    line.set_color( _linePrior, _priorColour )
                    line.set_style( _linePrior, f_getLineStyle( _priorStyle ) )
                    line.set_width( _linePrior, _activeWidth )

                // if not transfering, we need to delete before drawing new
                else

                    line.delete( _lineActive )

            // get our new active line set up
            _lineActive   := line.new(  _x1, _series, _x2, _series, _xloc,
                                  color     = _activeColour,
                                  style     = f_getLineStyle( _activeStyle ),
                                  width     = _activeWidth )

            // LABEL DRAWING

            // if our header label format input is not "DISABLED"
            if INP_labelHeaderFormat != i_header0

                // if we have a label, delete it before creating new
                if not na( _labelHeading )

                    label.delete( _labelHeading )

                // get our line header text string prepared
                string _string  = f_setNewlineOffset(
                                  f_getHeaderText( _verbose, _shorthand ),
                                  INP_labelLineOffset
                                  )

                // get our new heading label set up
                _labelHeading   := label.new( _x1, _series, _string, _xloc,
                                      color     = #00000000,
                                      style     = label.style_label_left,
                                      textcolor = _activeColour,
                                      size      = f_getLabelSize() )

            // if our price label format input is not "DISABLED"
            if INP_labelPriceFormat != i_price0

                // if we have a label, delete it before creating new
                if not na( _labelPrice )

                    label.delete( _labelPrice )

                // get our line price text string prepared
                string _string  = f_setNewlineOffset(
                                  f_getPriceText( _series ),
                                  INP_labelLineOffset
                                  )

                // get our new price label set up
                _labelPrice     := label.new( _x2, _series, _string, _xloc,
                                      color     = #00000000,
                                      style     = label.style_label_right,
                                      textcolor = _activeColour,
                                      size      = f_getLabelSize() )

////////////////////////////////////////////////////////////////////////////////
//                                                                            //
//                   ====== SERIES, LINES and LABELS ======                   //
//                                                                            //
////////////////////////////////////////////////////////////////////////////////

// -- timeframe change check
_changeTime = change( time( f_getResolution() ) ) != 0

// -- get our OHLC Series
[ _open,
  _high,
  _low,
  _close ] = security( syminfo.tickerid, f_getResolution(),
                      [ open[1], high[1], low[1], close[1] ],
                      false, true )

var float _previousOpen = na
var float _previousHigh = na
var float _previousLow = na
var float _previousClose = na

_previousOpen   := _changeTime ? _open   : _previousOpen[1]
_previousHigh   := _changeTime ? _high   : _previousHigh[1]
_previousLow    := _changeTime ? _low    : _previousLow[1]
_previousClose  := _changeTime ? _close  : _previousClose[1]

// -- sma for alert filtering
_alertSMA = sma( INP_alertSource, INP_alertLen )

////////////////////////////////////////////////////////////////////////////////
//                                                                            //
//                     ====== DRAWING and PLOTTING ======                     //
//                                                                            //
////////////////////////////////////////////////////////////////////////////////

f_renderLevel( INP_showOpenActive,  _previousOpen,  "Open",  "O" )

f_renderLevel( INP_showHighActive,  _previousHigh,  "High",  "H" )

f_renderLevel( INP_showLowActive,   _previousLow,   "Low",   "L" )

f_renderLevel( INP_showCloseActive, _previousClose, "Close", "C" )

////////////////////////////////////////////////////////////////////////////////
//                                                                            //
//                    ====== ALERTS FUNCTIONALITY ======                      //
//                                                                            //
////////////////////////////////////////////////////////////////////////////////

f_getCrossAlert( _showing, _level, _series, _direction ) =>

    var bool    _crossed    = false

    var label   _label      = na
    var label[] _crosses    = array.new_label(na)
    var label[] _crossesOld = array.new_label(na)

    // crossing booleans, over and under
    bool _over      = crossover( _series, _level ) ? true : false
    bool _under     = crossunder( _series, _level ) ? true : false

    if _changeTime
        if array.size( _crossesOld ) > 0
            for i = 1 to array.size( _crossesOld )
                label.delete( array.shift( _crossesOld ) )
        if array.size( _crosses ) > 0
            for i = 1 to array.size( _crosses )
                array.push( _crossesOld, array.shift( _crosses ) )
        _crossed := false
    else
        _crossed := _direction ? _over : _under

    // only print our X labels if user has enabled showing
    if INP_showTracker and _crossed
        if _showing
            _label := label.new( bar_index, _level,
                                 color  = _direction ? color.green : color.red,
                                 style  = label.style_xcross,
                                 size   = size.small )
            array.push( _crosses, _label )

    // return bool validation for the alertcondition()
    _return = _crossed

// -- only show plot if user enabled showing
plot( INP_showTracker ? _alertSMA : na )

_crossOverOpen  = f_getCrossAlert( INP_showOpenActive,  _previousOpen,  _alertSMA, true )
_crossOverHigh  = f_getCrossAlert( INP_showHighActive,  _previousHigh,  _alertSMA, true )
_crossOverLow   = f_getCrossAlert( INP_showLowActive,   _previousLow,   _alertSMA, true )
_crossOverClose = f_getCrossAlert( INP_showCloseActive, _previousClose, _alertSMA, true )

// -- OHLC Crossover Alert messaging setup
alertcondition( _crossOverOpen,  title = "PPL X-Over Open",  message = "PPL X Alert: SMA crossed over Previous Open Level")
alertcondition( _crossOverHigh,  title = "PPL X-Over High",  message = "PPL X Alert: SMA crossed over Previous High Level")
alertcondition( _crossOverLow,   title = "PPL X-Over Low",   message = "PPL X Alert: SMA crossed over Previous Low Level")
alertcondition( _crossOverClose, title = "PPL X-Over Close", message = "PPL X Alert: SMA crossed over Previous Close Level")

_crossUnderOpen  = f_getCrossAlert( INP_showOpenActive,  _previousOpen,  _alertSMA, false )
_crossUnderHigh  = f_getCrossAlert( INP_showHighActive,  _previousHigh,  _alertSMA, false )
_crossUnderLow   = f_getCrossAlert( INP_showLowActive,   _previousLow,   _alertSMA, false )
_crossUnderClose = f_getCrossAlert( INP_showCloseActive, _previousClose, _alertSMA, false )

// -- OHLC Crossunder Alert messaging setup
alertcondition( _crossUnderOpen,  title = "PPL X-Under Open",  message = "PPL X Alert: SMA crossed under Previous Open Level")
alertcondition( _crossUnderHigh,  title = "PPL X-Under High",  message = "PPL X Alert: SMA crossed under Previous High Level")
alertcondition( _crossUnderLow,   title = "PPL X-Under Low",   message = "PPL X Alert: SMA crossed under Previous Low Level")
alertcondition( _crossUnderClose, title = "PPL X-Under Close", message = "PPL X Alert: SMA crossed under Previous Close Level")

// ====== PEANUT
