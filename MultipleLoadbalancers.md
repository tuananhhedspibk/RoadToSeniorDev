# Multiple load balancers

Nguồn tham khảo: https://serverfault.com/questions/705194/is-it-possible-to-use-multiple-load-balancers-to-redirect-traffic-to-my-applicat

## Active/Passive Load balancers

Ta sẽ có 2 load balancers A và B, load balancer A sẽ chuyên xử lí các request đến từ IP1 chẳng hạn. Nếu trường hợp load balancer A bị sập thì Passive load balancer B sẽ thay thế

## Active/Active Load balancers

Ta có 2 load balancers A và B. Load balancer A sẽ chuyên xử lí các IP có đuôi lẻ, load balancer B sẽ chuyên xử lí các IP có đuôi chẵn.
