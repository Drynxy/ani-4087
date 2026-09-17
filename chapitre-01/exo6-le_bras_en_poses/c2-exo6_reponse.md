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

Quaternion Multiplier(const Quaternion& a, const Quaternion& b) {
    return {
        a.w*b.w - a.x*b.x - a.y*b.y - a.z*b.z,
        a.w*b.x + a.x*b.w + a.y*b.z - a.z*b.y,
        a.w*b.y - a.x*b.z + a.y*b.w + a.z*b.x,
        a.w*b.z + a.x*b.y - a.y*b.x + a.z*b.w
    };
}

Pose Composer(const Pose& parent, const Pose& local) {
    Vecteur p = Rotation(parent.rotation, local.position);

    return {
        {parent.position.x + p.x,
         parent.position.y + p.y,
         parent.position.z + p.z},
        Multiplier(parent.rotation, local.rotation)
    };
}

int main() {
    const double bras = 1.0;
    const double avantBras = 0.8;

    Pose epaule = {
        {0.0, 0.0, 0.0},
        {std::cos(M_PI / 4), 0.0, std::sin(M_PI / 4), 0.0}
    };

    Pose coudeLocal = {
        {bras, 0.0, 0.0},
        {1.0, 0.0, 0.0, 0.0}
    };

    Pose mainLocal = {
        {avantBras, 0.0, 0.0},
        {1.0, 0.0, 0.0, 0.0}
    };

    Pose coudeMonde = Composer(epaule, coudeLocal);
    Pose mainMonde = Composer(coudeMonde, mainLocal);

    std::cout << std::fixed << std::setprecision(4);
    std::cout << "Coude : "
              << coudeMonde.position.x << " "
              << coudeMonde.position.y << " "
              << coudeMonde.position.z << '\n';

    std::cout << "Main : "
              << mainMonde.position.x << " "
              << mainMonde.position.y << " "
              << mainMonde.position.z << '\n';

    // Vérification : l'épaule tourne de 90° autour de Y.
    epaule.rotation = {
        std::cos(M_PI / 4), 0.0, std::sin(M_PI / 4), 0.0
    };

    Pose coudeTourne = Composer(epaule, coudeLocal);
    Pose mainTournee = Composer(coudeTourne, mainLocal);

    std::cout << "Main après rotation de l'épaule : "
              << mainTournee.position.x << " "
              << mainTournee.position.y << " "
              << mainTournee.position.z << '\n';

    return 0;
}