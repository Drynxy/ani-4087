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
    Vecteur r;
    r.x = (1 - 2*q.y*q.y - 2*q.z*q.z) * p.x
       + (2*q.x*q.y - 2*q.z*q.w) * p.y
       + (2*q.x*q.z + 2*q.y*q.w) * p.z;

    r.y = (2*q.x*q.y + 2*q.z*q.w) * p.x
       + (1 - 2*q.x*q.x - 2*q.z*q.z) * p.y
       + (2*q.y*q.z - 2*q.x*q.w) * p.z;

    r.z = (2*q.x*q.z - 2*q.y*q.w) * p.x
       + (2*q.y*q.z + 2*q.x*q.w) * p.y
       + (1 - 2*q.x*q.x - 2*q.y*q.y) * p.z;

    return r;
}

Vecteur AppliquerPose(const Pose& pose, const Vecteur& point) {
    Vecteur r = Rotation(pose.rotation, point);
    return {
        r.x + pose.position.x,
        r.y + pose.position.y,
        r.z + pose.position.z
    };
}

Vecteur AppliquerTranslationPuisRotation(const Pose& pose, const Vecteur& point) {
    Vecteur t = {
        point.x + pose.position.x,
        point.y + pose.position.y,
        point.z + pose.position.z
    };
    return Rotation(pose.rotation, t);
}

void Afficher(const Vecteur& v) {
    std::cout << std::fixed << std::setprecision(4)
              << v.x << ' ' << v.y << ' ' << v.z << '\n';
}

int main() {
    Pose pose;
    Vecteur point;

    std::cin >> pose.position.x >> pose.position.y >> pose.position.z;
    std::cin >> pose.rotation.w >> pose.rotation.x
             >> pose.rotation.y >> pose.rotation.z;
    std::cin >> point.x >> point.y >> point.z;

    Afficher(AppliquerPose(pose, point));
    Afficher(AppliquerTranslationPuisRotation(pose, point));

    return 0;
}


Exemple où les deux résultats coïncident :

Pose :
position = (1, 2, 3)
rotation = (1, 0, 0, 0)

Point :
(4, 5, 6)

Les deux donnent :
5.0000 7.0000 9.0000

Pourquoi : le quaternion identité ne fait aucune rotation. L'ordre translation/rotation n'a donc aucun effet.