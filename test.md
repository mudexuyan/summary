test

struct Node {
    int size;
    int price;
    // 重载<运算符
    bool operator<(const Node &b) const {
        return this->size == b.size ? this->price > b.price : this->size < b.size;
    }
};

size:4 price:1
size:3 price:2
size:2 price:3
size:1 price:4
size:0 price:5

