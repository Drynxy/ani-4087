#include <iostream>
#include <iomanip>

struct Vecteur {
    double x, y, z;
};

struct Quaternion {
    double w, x, y, z;
};

struct Pose {
    Vecteur position;
    Quaternion rotation;
};

Vecteur Rotation(const Quaternion& q, const Vecteur& p) {
    return {
        (1 - 2*q.y*q.y - 2*q.z*q.z) * p.x
        + (2*q.x*q.y - 2*q.z*q.w) * p.y
        + (2*q.x*q.z + 2*q.y*q.w) * p.z,

        (2*q.x*q.y + 2*q.z*q.w) * p.x
        + (1 - 2*q.x*q.x - 2*q.z*q.z) * p.y
        + (2*q.y*q.z - 2*q.x*q.w) * p.z,

        (2*q.x*q.z - 2*q.y*q.w) * p.x
        + (2*q.y*q.z + 2*q.x*q.w) * p.y
        + (1 - 2*q.x*q.x - 2*q.y*q.y) * p.z
    };
}

Quaternion Multiplier(const Quaternion& a, const Quaternion& b) {
    return {
        a.w*b.w - a.x*b.x - a.y*b.y - a.z*b.z,
        a.w*b.x + a.x*b.w + a.y*b.z - a.z*b.y,
        a.w*b.y - a.x*b.z + a.y*b.w + a.z*b.x,
        a.w*b.z + a.x*b.y - a.y*b.x + a.z*b.w
    };
}

Pose Composer(const Pose& a, const Pose& b) {
    Quaternion q = Multiplier(a.rotation, b.rotation);
    Vecteur rb = Rotation(a.rotation, b.position);

    return {
        {rb.x + a.position.x,
         rb.y + a.position.y,
         rb.z + a.position.z},
        q
    };
}

Vecteur Appliquer(const Pose& pose, const Vecteur& point) {
    Vecteur r = Rotation(pose.rotation, point);
    return {
        r.x + pose.position.x,
        r.y + pose.position.y,
        r.z + pose.position.z
    };
}

void Afficher(const Vecteur& p) {
    std::cout << std::fixed << std::setprecision(4)
              << p.x << ' ' << p.y << ' ' << p.z << '\n';
}

int main() {
    Pose a, b;
    Vecteur point;

    std::cin >> a.position.x >> a.position.y >> a.position.z;
    std::cin >> a.rotation.w >> a.rotation.x >> a.rotation.y >> a.rotation.z;

    std::cin >> b.position.x >> b.position.y >> b.position.z;
    std::cin >> b.rotation.w >> b.rotation.x >> b.rotation.y >> b.rotation.z;

    std::cin >> point.x >> point.y >> point.z;

    Pose composee = Composer(a, b);

    Vecteur p1 = Appliquer(composee, point);
    Vecteur p2 = Appliquer(a, Appliquer(b, point));

    Vecteur ecart = {
        p1.x - p2.x,
        p1.y - p2.y,
        p1.z - p2.z
    };

    Afficher(p1);
    Afficher(p2);
    Afficher(ecart);

    return 0;
}