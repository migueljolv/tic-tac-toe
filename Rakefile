task :default => [:test]

# We play 3x3 tic-tac-toe here.
d = [3, 3]

task :test do
  # Look at the game file
  f = IO.read 'README.md'
  # Find and parse all games
  games = f.scan(
    # does it look like a correctly-sized table?
    /(?:^[ \t]*\|?(?:[^|]+\|){#{d[0]-1}}[^|]+\|?[ \t]*\n){#{d[1]+1}}(?:\z|^[^|\n]*\n)/
  ).grep(
    # does it have the telltale second-line header-bar?
    /\A.*\n[ \t]*\|?(?:[ \t]*:?---+:?[ \t]*\|){#{d[0]-1}}[ \t]*:?---+:?[ \t]*\|?[ \t]*\n/
  ).map{ |s|
    # split into lines
    rows = s.split /\n/
    # discard the second (header-bar) line
    rows.delete_at 1
    # process the rows
    rows.map!{ |row|
      # split on bars
      cols = row.split /\|/
      # remove extra field due to leading bar
      cols.shift if cols.size > d[0]
      # remove extra field due to trailing bar
      cols[0..d[0]-1].map{ |cell|
        # sanitize content
        cell =~ /^\s*x\s$/i ? 'x' : cell =~ /^\s*o\s$/i ? 'o' : ' '
      }
    }
  }
  # Anylize all games
  results = games.map{ |game|
    # precompute transposed game
    tgame = game.transpose
    # illegal states: o's > x's, x's > o's + 1
    os = game.flatten.count 'o'
    xs = game.flatten.count 'x'
    if os > xs then
      {err: true, msg: 'O took too many turns'}
    elsif xs > os+1 then
      {err: true, msg: 'X took too many turns'}
    else
      # possible wins:
      ow = 0
      xw = 0
      # any row
      ow += game.count{|row| row.count('o') == d[0] }
      xw += game.count{|row| row.count('x') == d[0] }
      # any colulm
      ow += tgame.count{|col| col.count('o') == d[1] }
      xw += tgame.count{|col| col.count('x') == d[1] }
      # any diagonal
      # use transposed game if h>w (so we are guaranteed that w>=h)
      dgame = d[1] > d[0] ? tgame : game
      # for non-square grids, there are >2 diagonals...
      ow += (d.max - d.min + 1).times.count{ |i| d.min.times.count{ |n| dgame[i+n][n] == 'o' } == d.min }
      xw += (d.max - d.min + 1).times.count{ |i| d.min.times.count{ |n| dgame[i+n][n] == 'x' } == d.min }
      # return win info
      if ow > 0 && xw > 0 then
        {err: true, msg: 'both X and O occupy winning positions'}
      elsif os + xs == game.flatten.size
        {err: false, tie: true }
      elsif xw > 0 || ow > 0
        {err: false, win: (xw > 0 ? 'x' : 'o'), num: (xw > 0 ? xw : ow) }
      else
        {err: false }
      end
    end
  }
  # Print summary and analyses
  puts "Details:"
  puts "========"
  games.each_index{ |i|
    puts "Game \##{i+1} was won by #{results[i][:win].upcase} in the #{ordinal(results[i][:num])} degree" if results[i][:win]
    puts "Game \##{i+1} was tied" if results[i][:tie]
    puts "Game \##{i+1} is invalid because #{results[i][:msg]}" if results[i][:err]
    puts "Game \##{i+1} is unfinished" if !results[i][:err] && !results[i][:win] && !results[i][:tie]
  }
  puts ""
  puts "Summary:"
  puts "========"
  puts "Total games: #{games.size}"
  puts "Invalid: #{results.count{|r| r[:err]}}"
  puts "Unfinished: #{results.count{|r| !r[:err] && !r[:win] && !r[:tie]}}"
  puts "Won by X: #{results.count{|r| r[:win] == 'x'}}"
  puts "Won by O: #{results.count{|r| r[:win] == 'o'}}"
  puts "Tied: #{results.count{|r| r[:tie]}}"
  puts ""

  raise results.find{|r| r[:err]}[:msg] if results.any?{|r| r[:err]}
end

def ordinal n
  n.to_s + ((11..13).include?(n % 100) ? 'th' : %w(th st nd rd th)[[n % 10, 4].min])
end
