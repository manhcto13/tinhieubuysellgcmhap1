# tinhieubuysellgcmhap1
//@version=5
indicator(title="GCM Heikin Ashi with Pivots", shorttitle="GCM HAP", overlay=false, format=format.price, precision=2)

// --- Group 1: Pivot Detection Inputs ---
string g1 = "Pivot Detection"
leftBars = input.int(10, title="Left Bars (Strength)", minval=1, group=g1)
rightBars = input.int(5, title="Right Bars (Confirmation)", minval=1, tooltip="The number of bars to the right required to confirm a pivot. This introduces a delay. A label will appear 'Right Bars' after the pivot has formed.", group=g1)

// --- Group 2: Volume Spike Detection (VROC) ---
string g2 = "Volume Spike Detection (VROC)"
enableVroc = input.bool(true, title="Enable Volume Spike Highlighting", group=g2)
vrocLength = input.int(14, title="VROC Length", minval=1, group=g2)
vrocThreshold = input.float(50, title="VROC Threshold %", minval=0, group=g2, tooltip="Volume must increase by more than this percentage to be highlighted.")
var color vrocHighlightColor = #2dff00

// --- Group 3: Label Customization Inputs ---
string g3 = "Label Customization"
var color defaultHighColor = #801922
var color defaultLowColor  = #1b5e20
label_text_mode = input.string("High & Low", title="Label Text Mode", options=["High & Low", "Buy & Sell"], group=g3)
h_label_color = input.color(defaultHighColor, title="High Label Color", group=g3, inline="h_label")
h_text_color = input.color(color.new(color.white, 0), title="Text Color", group=g3, inline="h_label")
h_label_style_str = input.string("Box Down", title="High Label Style", options=["Box Down", "Circle", "Diamond", "Square"], group=g3)
l_label_color = input.color(defaultLowColor, title="Low Label Color", group=g3, inline="l_label")
l_text_color = input.color(color.new(color.white, 0), title="Text Color", group=g3, inline="l_label")
l_label_style_str = input.string("Box Up", title="Low Label Style", options=["Box Up", "Circle", "Diamond", "Square"], group=g3)
label_size_str = input.string("Tiny", title="Label Size", options=["Tiny", "Small", "Normal", "Large", "Huge"], group=g3)

// --- Heikin Ashi Calculation ---
ha_ticker = ticker.heikinashi(syminfo.tickerid)
[ha_open, ha_high, ha_low, ha_close] = request.security(ha_ticker, timeframe.period, [open, high, low, close], lookahead=barmerge.lookahead_off)

// --- VROC Calculation ---
volumeRoc = ta.roc(volume, vrocLength)
isVrocSpike = enableVroc and (volumeRoc > vrocThreshold) and (close > open)

// --- Determine Final Candle Color ---
final_ha_color = isVrocSpike ? vrocHighlightColor : (ha_close >= ha_open ? color.new(color.green, 0) : color.new(color.red, 0))
plotcandle(ha_open, ha_high, ha_low, ha_close, title="Heikin Ashi Candles", color=final_ha_color, wickcolor=final_ha_color, bordercolor=final_ha_color)

// --- Pivot High/Low Logic ---
pivotHighValue = ta.pivothigh(high, leftBars, rightBars)
pivotLowValue = ta.pivotlow(low, leftBars, rightBars)

// --- Convert String Inputs to Pine Script Constants ---
string high_text_to_plot = label_text_mode == "High & Low" ? "H" : "S"
string low_text_to_plot  = label_text_mode == "High & Low" ? "L" : "B"

// CORRECTED: Expanded switch statements to proper multi-line format
var string high_style = switch h_label_style_str
    "Circle"    => label.style_circle
    "Diamond"   => label.style_diamond
    "Square"    => label.style_square
    => label.style_label_down

var string low_style = switch l_label_style_str
    "Circle"    => label.style_circle
    "Diamond"   => label.style_diamond
    "Square"    => label.style_square
    => label.style_label_up 

var string label_size = switch label_size_str
    "Small"     => size.small
    "Normal"    => size.normal
    "Large"     => size.large
    "Huge"      => size.huge
    => size.tiny

// --- Plot H/L Labels at Confirmed Pivots ---
if not na(pivotHighValue)
    label.new(bar_index[rightBars], ha_high[rightBars], text=high_text_to_plot, color=h_label_color, textcolor=h_text_color, style=high_style, yloc=yloc.price, size=label_size)
if not na(pivotLowValue)
    label.new(bar_index[rightBars], ha_low[rightBars], text=low_text_to_plot, color=l_label_color, textcolor=l_text_color, style=low_style, yloc=yloc.price, size=label_size)

