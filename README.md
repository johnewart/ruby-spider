
Spider, a Web spidering library for Ruby. It handles the robots.txt,
scraping, collecting, and looping so that you can just handle the data.

== Examples

=== Crawl the Web, loading each page in turn, until you run out of memory

 require 'spider'
 Spider.start_at('http://mike-burns.com/') {}

=== To handle erroneous responses

 require 'spider'
 Spider.start_at('http://mike-burns.com/') do |s|
   s.on :failure do |a_url, resp, prior_url|
     puts "URL failed: #{a_url}"
     puts " linked from #{prior_url}"
   end
 end

=== Or handle successful responses

 require 'spider'
 Spider.start_at('http://mike-burns.com/') do |s|
   s.on :success do |a_url, resp, prior_url|
     puts "#{a_url}: #{resp.code}"
     puts resp.body
     puts
   end
 end

=== Limit to just one domain

 require 'spider'
 Spider.start_at('http://mike-burns.com/') do |s|
   s.add_url_check do |a_url|
     a_url =~ %r{^http://mike-burns.com.*}
   end
 end

=== Pass headers to some requests

 require 'spider'
 Spider.start_at('http://mike-burns.com/') do |s|
   s.setup do |a_url|
     if a_url =~ %r{^http://.*wikipedia.*}
       headers['User-Agent'] = "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
     end
   end
 end

=== Use memcached to track cycles

 require 'spider'
 require 'spider/included_in_memcached'
 SERVERS = ['10.0.10.2:11211','10.0.10.3:11211','10.0.10.4:11211']
 Spider.start_at('http://mike-burns.com/') do |s|
   s.check_already_seen_with IncludedInMemcached.new(SERVERS)
 end

=== Track cycles with a custom object

 require 'spider'
 class ExpireLinks < Hash
   def <<(v)
     self[v] = Time.now
   end
   def include?(v)
     self[v].kind_of?(Time) && (self[v] + 86400) >= Time.now
   end
 end

 Spider.start_at('http://mike-burns.com/') do |s|
   s.check_already_seen_with ExpireLinks.new
 end

=== Store nodes to visit with Amazon SQS

 require 'spider'
 require 'spider/next_urls_in_sqs'
 Spider.start_at('http://mike-burns.com') do |s|
   s.store_next_urls_with NextUrlsInSQS.new(AWS_ACCESS_KEY, AWS_SECRET_ACCESS_KEY)
 end

==== Store nodes to visit with a custom object

 require 'spider'
 class MyArray < Array
   def pop
	super
   end
  
   def push(a_msg)
     super(a_msg)
   end
 end

 Spider.start_at('http://mike-burns.com') do |s|
   s.store_next_urls_with MyArray.new
 end

=== Create a URL graph

 require 'spider'
 nodes = {}
 Spider.start_at('http://mike-burns.com/') do |s|
   s.add_url_check {|a_url| a_url =~ %r{^http://mike-burns.com.*} }

   s.on(:every) do |a_url, resp, prior_url|
     nodes[prior_url] ||= []
     nodes[prior_url] << a_url
   end
 end

=== Use a proxy

 require 'net/http_configuration'
 require 'spider'
 http_conf = Net::HTTP::Configuration.new(:proxy_host => '7proxies.org',
                                          :proxy_port => 8881)  
 http_conf.apply do
   Spider.start_at('http://img.4chan.org/b/') do |s|
     s.on(:success) do |a_url, resp, prior_url|
       File.open(a_url.gsub('/',':'),'w') do |f|
         f.write(resp.body)
       end
     end
   end
 end

== Author

John Ewart <john@unixninjas.org>

John Nagro john.nagro@gmail.com

Mike Burns http://mike-burns.com mike@mike-burns.com (original author)

Many thanks to:
Matt Horan
Henri Cook
Sander van der Vliet
John Buckley
Brian Campbell

With `robot_rules' from James Edward Gray II via
http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-talk/177589
