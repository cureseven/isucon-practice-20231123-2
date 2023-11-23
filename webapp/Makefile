.PHONY: *

gogo: stop-services truncate-logs start-services bench

stop-services:
	sudo systemctl stop nginx
	sudo systemctl stop isuports.service
	ssh isucon-s3 "sudo systemctl stop mysql"

start-services:
	ssh isucon-s3 "sudo systemctl start mysql"
	sudo systemctl start isuports.service
	sleep 5
	sudo systemctl start nginx

truncate-logs:
	sudo truncate --size 0 /var/log/nginx/access.log
	sudo truncate --size 0 /var/log/nginx/error.log
	ssh isucon-s3 "sudo truncate --size 0 /var/log/mysql/mysql-slow.log"
	ssh isucon-s3 "sudo chmod 777 /var/log/mysql/mysql-slow.log"
	sudo journalctl --vacuum-size=1K

bench:
	ssh isucon-bench "cd ~/bench && ./bench -target-addr isucon-s1:443"
kataribe:
	cd ~/ && sudo cat /var/log/nginx/access.log | ./kataribe

pprof: TIME=60
pprof: PROF_FILE=~/pprof.samples.$(shell TZ=Asia/Tokyo date +"%H%M").$(shell git rev-parse HEAD | cut -c 1-8).pb.gz
pprof:
	curl -sSf "http://localhost:6060/debug/fgprof?seconds=$(TIME)" > $(PROF_FILE)
	go tool pprof $(PROF_FILE)
