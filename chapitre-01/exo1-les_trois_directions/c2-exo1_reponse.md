#include <iostream>
#include <iomanip>

struct Vecteur {
    double x, y, z;
};

Vecteur Avant() {
    return {0.0, 0.0, -1.0};
}

Vecteur Haut() {
    return {0.0, 1.0, 0.0};
}

Vecteur Droite() {
    return {1.0, 0.0, 0.0};
}

double ProduitScalaire(const Vecteur& a, const Vecteur& b) {
    return a.x * b.x + a.y * b.y + a.z * b.z;
}

int main() {
    Vecteur p;
    std::cin >> p.x >> p.y >> p.z;

    std::cout << std::fixed << std::setprecision(4);
    std::cout << ProduitScalaire(p, Avant()) << '\n';
    std::cout << ProduitScalaire(p, Haut()) << '\n';
    std::cout << ProduitScalaire(p, Droite()) << '\n';

    return 0;
}