
docker run -p 11211:11211 --name lele_memcache memcached

./mutilate -s localhost:11211 -T 24 -c 8 -t 200

top -Hp 120333