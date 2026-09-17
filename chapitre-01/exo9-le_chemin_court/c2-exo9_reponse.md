#include <iostream>
#include <iomanip>
#include <cmath>

struct Quaternion {
    double w, x, y, z;
};

struct Vecteur {
    double x, y, z;
};

Quaternion Conjugue(const Quaternion& q) {
    return {q.w, -q.x, -q.y, -q.z};
}

Quaternion Multiplier(const Quaternion& a, const Quaternion& b) {
    return {
        a.w*b.w - a.x*b.x - a.y*b.y - a.z*b.z,
        a.w*b.x + a.x*b.w + a.y*b.z - a.z*b.y,
        a.w*b.y - a.x*b.z + a.y*b.w + a.z*b.x,
        a.w*b.z + a.x*b.y - a.y*b.x + a.z*b.w
    };
}

Vecteur VitesseAngulaire(Quaternion q1, Quaternion q2, double dt,
                         bool cheminCourt) {
    double dot = q1.w*q2.w + q1.x*q2.x + q1.y*q2.y + q1.z*q2.z;

    if (cheminCourt && dot < 0.0) {
        q2.w = -q2.w;
        q2.x = -q2.x;
        q2.y = -q2.y;
        q2.z = -q2.z;
    }

    Quaternion delta = Multiplier(Conjugue(q1), q2);

    double w = std::max(-1.0, std::min(1.0, delta.w));
    double angle = 2.0 * std::acos(w);
    double s = std::sin(angle * 0.5);

    if (std::abs(s) < 1e-12)
        return {0.0, 0.0, 0.0};

    return {
        delta.x / s * angle / dt,
        delta.y / s * angle / dt,
        delta.z / s * angle / dt
    };
}

void Afficher(const Vecteur& v) {
    std::cout << std::fixed << std::setprecision(4)
              << v.x << ' ' << v.y << ' ' << v.z << '\n';
}

int main() {
    double angle = M_PI / 180.0;
    Quaternion q1 = {std::cos(angle), 0.0, 0.0, std::sin(angle)};
    Quaternion q2 = {-std::cos(angle), 0.0, 0.0, -std::sin(angle)};
    double dt = 1.0;

    Afficher(VitesseAngulaire(q1, q2, dt, true));
    Afficher(VitesseAngulaire(q1, q2, dt, false));

    return 0;
}

Résultat :

0.0000 0.0000 0.0000
0.0000 0.0000 6.2483