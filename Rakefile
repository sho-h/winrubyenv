require 'net/http'
require 'uri'
require 'tempfile'

class Redirect < StandardError; end

task default: :generate

desc "generate rubyenv"
task "generate", ['version'] do |task, args|
  version = args.version || "2.3.3"
  url = "https://dl.bintray.com/oneclick/rubyinstaller/ruby-#{version}-x64-mingw32.7z"
  max_redirect = 5
  redirect_count = 0
  f = Tempfile.open(["ruby", ".7z"])
  f.binmode
  temp_path = nil
  begin
    uri = URI.parse(url)
    Net::HTTP.start(uri.host, uri.port, use_ssl: uri.scheme == "https", ca_file: "./cacert.pem") do |http|
      case res = http.get(uri.request_uri)
      when Net::HTTPSuccess
        f.write(res.body)
      when Net::HTTPRedirection
        raise Redirect.new("redirected: to=<#{url = res['Location']}> count=<#{redirect_count += 1}>")
      end
    end
    temp_path = f.path.dup
    f.close(false)
    system("7z", "x", temp_path, "-o./ruby_env/")
    f.close(true)
  rescue Redirect => e
    # puts e.message
    retry if redirect_count <= max_redirect
    puts "too many redirect. exit: retry=<#{redirect_count}>"
    exit(1)
  rescue => e
    p e.message
    puts e.backtrace
  end
  File.open("./cmd.bat.template") do |template|
    File.open("./ruby_env/cmd.bat", "w") do |f|
      f.write(template.read.gsub(/%rubyver%/, version))
    end
  end
end