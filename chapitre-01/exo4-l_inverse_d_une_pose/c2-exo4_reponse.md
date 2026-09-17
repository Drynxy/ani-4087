#include <iostream>
#include <iomanip>
#include <cmath>

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

Pose Inverser(const Pose& pose) {
    Quaternion qc = {
        pose.rotation.w,
        -pose.rotation.x,
        -pose.rotation.y,
        -pose.rotation.z
    };

    Vecteur positionOpposee = {
        -pose.position.x,
        -pose.position.y,
        -pose.position.z
    };

    return {
        Rotation(qc, positionOpposee),
        qc
    };
}

Vecteur AppliquerPose(const Pose& pose, const Vecteur& point) {
    Vecteur r = Rotation(pose.rotation, point);
    return {
        r.x + pose.position.x,
        r.y + pose.position.y,
        r.z + pose.position.z
    };
}

int main() {
    Pose pose;
    Vecteur point;

    std::cin >> pose.position.x >> pose.position.y >> pose.position.z;
    std::cin >> pose.rotation.w >> pose.rotation.x
             >> pose.rotation.y >> pose.rotation.z;
    std::cin >> point.x >> point.y >> point.z;

    Vecteur transforme = AppliquerPose(pose, point);
    Vecteur retour = AppliquerPose(Inverser(pose), transforme);

    Vecteur ecart = {
        retour.x - point.x,
        retour.y - point.y,
        retour.z - point.z
    };

    std::cout << std::fixed << std::setprecision(4)
              << ecart.x << ' ' << ecart.y << ' ' << ecart.z << '\n';

    return 0;
}