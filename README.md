# Celestia-Light-Node

1/ Cập nhật phiên bản mới:

    sudo apt update && sudo apt upgrade -y
    
2/ Bổ sung package:

    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y
    
3/ Cài đặt Golang:

    ver="1.19.1" 
    cd $HOME 
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" 
    
    
    sudo rm -rf /usr/local/go 
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" 
    rm "go$ver.linux-amd64.tar.gz"

Chuyển thư mục Golang:

    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
    source $HOME/.bash_profile

Kiểm tra phiên bản:

    go version
    
4/ Cài đặt Light Node:

    cd $HOME 
    rm -rf celestia-node 
    git clone https://github.com/celestiaorg/celestia-node.git

    cd celestia-node/ 
    git checkout tags/v0.8.0
    make build 

    make install 
    
    make cel-key
    
5/ Cài đặt mạng:

    celestia light init --p2p.network blockspacerace 
 
Chạy node bằng RPC, dùng port 26657. Đổi địa chỉ ip-addr = địa chỉ VPS

    celestia light start --core.ip https://rpc-blockspacerace.pops.one/ --gateway --gateway.addr <ip-address> --gateway.port 26657 --p2p.network blockspacerace
    
Chạy node bằng port 26659, thay ip-addr = địa chỉ VPS

    celestia light start --p2p.network blockspacerace --core.ip <ip-address> --gateway --gateway.addr <ip-address> --gateway.port 26659
    
6/ Tạo ví mới:

    ./cel-key add wallet --keyring-backend test --node.type light --p2p.network blockspacerace
    
Nếu đã có ví, sử dụng lệnh khôi phục:

    ./cel-key add wallet --recover --keyring-backend test --node.type light --p2p.network blockspacerace
    
Muốn kiểm tra các ví đang chạy trên node của bạn:

    ./cel-key list --node.type light --keyring-backend test --p2p.network blockspacerace
    
***** Nhớ Faucet token vào địa chỉ ví bạn đang chạy node *****

7/ Có 2 lựa chọn để chạy Node: Bằng Key của bạn hoặc dùng SystemD (chỉ nên chọn 1)

Chạy node bằng Key: thay ip address = địa chỉ IP VPS

    celestia light start --core.ip <ip-address> --keyring.accname wallet --gateway --gateway.addr <ip-address> --gateway.port 26659 --p2p.network blockspacerace
    
Chạy bằng systemD; cần khởi tạo service như dưới đây:

    sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-lightd.service
    [Unit]
    Description=celestia-lightd Light Node
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=/usr/local/bin/celestia light start --core.ip https://rpc-blockspacerace.pops.one --core.rpc.port 26657 --core.grpc.port 9090 --      keyring.accname my_celes_key --metrics.tls=false --metrics --metrics.endpoint otel.celestia.tools:4318 --gateway --gateway.addr localhost --gateway.port 26659 --p2p.network blockspacerace
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=4096

    [Install]
    WantedBy=multi-user.target
    EOF

Lưu trữ hệ thống:

    cat /etc/systemd/system/celestia-lightd.service
    
Khởi chạy hệ thống:

    sudo systemctl enable celestia-lightd
    sudo systemctl start celestia-lightd
    sudo journalctl -u celestia-lightd.service -f

Lấy node ID, nhập lệnh này:

    curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658

Chúc các bạn thành công!

Cộng đồng chạy Node & Validator VietNam, nơi thảo luận và chia sẻ kinh nghiệm về chạy node/validator.

    Chanel: https://t.me/RunnodeVietNamese
    Youtube: https://www.youtube.com/@nodevalidatorvietnam
