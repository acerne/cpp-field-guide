[← C++ Field Guide](../README.md)

# Basic Numeric Operations Are Not constexpr

Standard numeric operations, such as `std::sqrt`, `std::pow`, and `std::log` are not `constexpr`. They cannot be used at compile time.

Supposedly C++26 will fix this and make them `constexpr`. The future is here, boys.