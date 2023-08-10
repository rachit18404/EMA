//@version=5
strategy("Buy on Rejection after Lowest Low Pullback with EMA Stop Loss", overlay=true)

// Input for EMA length
ema_length = input(9, title="EMA Length")

// Offset for stop loss
stop_loss_offset = input.float(0.5, title="Stop Loss Offset")


// Number of candles to check for pullback
pullback_candles = input(3, title="Pullback Candles")

// Calculate EMA
ema = ta.ema(close, ema_length)

// Profit target offset
profit_target_offset = input.float(1.5, title="Profit Target Offset")


// Calculate profit target for long positions
long_profit_target = math.max(ema, high) + profit_target_offset
short_profit_target = math.min(ema, low) - profit_target_offset

// Calculate the number of bars in the last 7 days
start_timestamp = timestamp("2023-07-01T00:00:00")  // Replace with your desired starting timestamp
bars_in_7_days = ta.barssince(time >= start_timestamp)

// Lowest low pullback conditions
pullback_low = close < ema
lowest_low = high
for i = 1 to pullback_candles
    if low[i] < lowest_low
        lowest_low := low[i]

// Bullish and bearish rejection conditions
bullish_rejection = close > ema and low < ema and close > lowest_low and low < lowest_low
bearish_rejection = close < ema and high > ema and close < lowest_low and high > lowest_low

// Strategy conditions
long_condition = bullish_rejection and bar_index > bars_in_7_days
short_condition = bearish_rejection and bar_index > bars_in_7_days

// Strategy orders
strategy.entry("Long", strategy.long, when=long_condition)
strategy.entry("Short", strategy.short, when=short_condition)

// Calculate stop loss for long positions
long_stop_loss = math.min(ema, low)- stop_loss_offset
short_stop_loss=math.max(ema,high)+ stop_loss_offset
// Set stop loss level for long positions
strategy.exit("Long Stop Loss", from_entry="Long", stop=long_stop_loss, limit=long_profit_target)

// Set stop loss level for short positions
strategy.exit("Short Stop Loss", from_entry="Short", stop=short_stop_loss, limit=short_profit_target)


// Plot EMA on the chart
plot(ema, color=color.blue, title="9 EMA")
