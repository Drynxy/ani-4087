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

Vecteur AppliquerPose(const Pose& pose, const Vecteur& point) {
    const auto& q = pose.rotation;

    Vecteur r;
    r.x = (1 - 2*q.y*q.y - 2*q.z*q.z) * point.x
       + (2*q.x*q.y - 2*q.z*q.w) * point.y
       + (2*q.x*q.z + 2*q.y*q.w) * point.z;

    r.y = (2*q.x*q.y + 2*q.z*q.w) * point.x
       + (1 - 2*q.x*q.x - 2*q.z*q.z) * point.y
       + (2*q.y*q.z - 2*q.x*q.w) * point.z;

    r.z = (2*q.x*q.z - 2*q.y*q.w) * point.x
       + (2*q.y*q.z + 2*q.x*q.w) * point.y
       + (1 - 2*q.x*q.x - 2*q.y*q.y) * point.z;

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

    Vecteur resultat = AppliquerPose(pose, point);

    std::cout << std::fixed << std::setprecision(4)
              << resultat.x << ' '
              << resultat.y << ' '
              << resultat.z << '\n';

    return 0;
}