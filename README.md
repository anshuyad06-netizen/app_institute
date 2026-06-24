#include "imgui.h"
#include "imgui_impl_glfw.h"
#include "imgui_impl_opengl3.h"

#include <GLFW/glfw3.h>
#include <cmath>
#include <vector>
#include <string>
#include <fstream>
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"

// =======================
// SAVE STUDENTS
// =======================

void SaveStudents(const std::vector<std::string>& students)
{
    std::ofstream file("students.txt");

    for (const auto& student : students)
    {
        file << student << std::endl;
    }
}

// =======================
// LOAD STUDENTS
// =======================

void LoadStudents(std::vector<std::string>& students)
{
    std::ifstream file("students.txt");

    std::string student;

    while (getline(file, student))
    {
        students.push_back(student);
    }
}

// =======================
// MAIN
// =======================

int main()
{
    // GLFW INIT
    if (!glfwInit())
        return -1;

    // WINDOW
    GLFWwindow* window = glfwCreateWindow(
        1280,
        720,
        "SchoolOS",
        NULL,
        NULL
    );

    if (window == NULL)
        return -1;

    glfwMakeContextCurrent(window);

    glfwSwapInterval(1);

    // IMGUI
    IMGUI_CHECKVERSION();

    ImGui::CreateContext();

    ImGuiIO& io = ImGui::GetIO();

    (void)io;

    ImGui::StyleColorsDark();

    ImGui_ImplGlfw_InitForOpenGL(window, true);

    ImGui_ImplOpenGL3_Init("#version 130");
// =======================
// LOAD GRASS TEXTURE
// =======================

int width, height, nrChannels;

unsigned char* data =
    stbi_load(
        "grass.jpg",
        &width,
        &height,
        &nrChannels,
        0
    );

glGenTextures(1, &grassTexture);

glBindTexture(
    GL_TEXTURE_2D,
    grassTexture
);

glTexParameteri(
    GL_TEXTURE_2D,
    GL_TEXTURE_WRAP_S,
    GL_REPEAT
);

glTexParameteri(
    GL_TEXTURE_2D,
    GL_TEXTURE_WRAP_T,
    GL_REPEAT
);

glTexParameteri(
    GL_TEXTURE_2D,
    GL_TEXTURE_MIN_FILTER,
    GL_LINEAR
);

glTexParameteri(
    GL_TEXTURE_2D,
    GL_TEXTURE_MAG_FILTER,
    GL_LINEAR
);

if (data)
{
    glTexImage2D(
        GL_TEXTURE_2D,
        0,
        GL_RGB,
        width,
        height,
        0,
        GL_RGB,
        GL_UNSIGNED_BYTE,
        data
    );

    glGenerateMipmap(GL_TEXTURE_2D);
}

stbi_image_free(data);

    // =======================
    // VARIABLES
    // =======================

    char username[128] = "";

    char password[128] = "";

    bool loggedIn = false;

    int currentPage = 0;

    std::vector<std::string> students;

    LoadStudents(students);

    char studentName[128] = "";

    // 3D
    float rotation = 0.0f;
 float cloudMove = 0.0f;
    float cameraX = 0.0f;
    float cameraY = 0.0f;
    float cameraZ = -8.0f;
    GLuint grassTexture;
float yaw = 0.0f;
float pitch = 0.0f;

double lastMouseX = 640;
double lastMouseY = 360;

    // =======================
    // MAIN LOOP
    // =======================

    while (!glfwWindowShouldClose(window))
    {
        glfwPollEvents();
double mouseX, mouseY;

glfwGetCursorPos(
    window,
    &mouseX,
    &mouseY
);

float mouseSpeed = 0.05f;

yaw += (mouseX - lastMouseX) * mouseSpeed;
pitch += (mouseY - lastMouseY) * mouseSpeed;

lastMouseX = mouseX;
lastMouseY = mouseY;
        // =======================
        // CAMERA MOVEMENT
        // =======================

        if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
        {
            cameraZ += 0.1f;
        }

        if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
        {
            cameraZ -= 0.1f;
        }

        if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
        {
            cameraX += 0.1f;
        }

        if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
        {
            cameraX -= 0.1f;
        }

        // =======================
        // IMGUI FRAME
        // =======================

        ImGui_ImplOpenGL3_NewFrame();

        ImGui_ImplGlfw_NewFrame();

        ImGui::NewFrame();

        // =======================
        // UI WINDOW
        // =======================

        ImGui::Begin("SchoolOS");

        // LOGIN
        if (!loggedIn)
        {
            ImGui::Text("Login System");

            ImGui::InputText(
                "Username",
                username,
                IM_ARRAYSIZE(username)
            );

            ImGui::InputText(
                "Password",
                password,
                IM_ARRAYSIZE(password),
                ImGuiInputTextFlags_Password
            );

            if (ImGui::Button("Login"))
            {
                loggedIn = true;
            }
        }

        // DASHBOARD
        else
        {
            // SIDEBAR
            ImGui::BeginChild(
                "Sidebar",
                ImVec2(200, 0),
                true
            );

            if (ImGui::Button("Home", ImVec2(180, 40)))
                currentPage = 0;

            if (ImGui::Button("Students", ImVec2(180, 40)))
                currentPage = 1;

            if (ImGui::Button("Teachers", ImVec2(180, 40)))
                currentPage = 2;

            if (ImGui::Button("Fees", ImVec2(180, 40)))
                currentPage = 3;

            if (ImGui::Button("Attendance", ImVec2(180, 40)))
                currentPage = 4;

            ImGui::EndChild();

            ImGui::SameLine();

            // CONTENT
            ImGui::BeginChild(
                "Content",
                ImVec2(0, 0),
                true
            );

            // HOME
            if (currentPage == 0)
            {
                ImGui::Text("Home Dashboard");

                ImGui::Separator();

                ImGui::Text(
                    "Welcome to SchoolOS Admin Panel"
                );
            }

            // STUDENTS
            if (currentPage == 1)
            {
                ImGui::Text("Student Management");

                ImGui::Separator();

                ImGui::InputText(
                    "Student Name",
                    studentName,
                    IM_ARRAYSIZE(studentName)
                );

                // ADD
                if (ImGui::Button("Add Student"))
                {
                    students.push_back(studentName);

                    SaveStudents(students);

                    studentName[0] = '\0';
                }

                ImGui::Separator();

                ImGui::Text("Student List:");

                // LIST
                for (int i = 0; i < students.size(); i++)
                {
                    ImGui::Text(
                        "%s",
                        students[i].c_str()
                    );

                    ImGui::SameLine();

                    std::string buttonLabel =
                        "Delete##" + std::to_string(i);

                    // DELETE
                    if (ImGui::Button(buttonLabel.c_str()))
                    {
                        students.erase(
                            students.begin() + i
                        );

                        SaveStudents(students);

                        i--;
                    }
                }
            }

            // TEACHERS
            if (currentPage == 2)
            {
                ImGui::Text("Teachers Section");
            }

            // FEES
            if (currentPage == 3)
            {
                ImGui::Text("Fees Section");
            }

            // ATTENDANCE
            if (currentPage == 4)
            {
                ImGui::Text("Attendance Section");
            }

            ImGui::EndChild();
        }

        ImGui::End();

        // =======================
        // RENDERING
        // =======================

        ImGui::Render();

        int display_w, display_h;

        glfwGetFramebufferSize(
            window,
            &display_w,
            &display_h
        );

        glViewport(
            0,
            0,
            display_w,
            display_h
        );

        // SKY
        glClearColor(
            0.02f,
            0.02f,
            0.08f,
            1.0f
        );

        // CLEAR
        glClear(
            GL_COLOR_BUFFER_BIT |
            GL_DEPTH_BUFFER_BIT
        );

        // DEPTH
        glEnable(GL_DEPTH_TEST);
        glEnable(GL_TEXTURE_2D);
        // ROTATION
        rotation += 0.5f;
        cloudMove += 0.01f;
        // PROJECTION
        glMatrixMode(GL_PROJECTION);

        glLoadIdentity();
        
glRotatef(pitch, 1.0f, 0.0f, 0.0f);

glRotatef(yaw, 0.0f, 1.0f, 0.0f);

        float aspect =
            (float)display_w /
            (float)display_h;

        glFrustum(
            -aspect,
            aspect,
            -1.0,
            1.0,
            1.5,
            100.0
        );

        // MODEL VIEW
        glMatrixMode(GL_MODELVIEW);

        glLoadIdentity();

        // CAMERA
        glTranslatef(
            cameraX,
            cameraY,
            cameraZ
        );

      // =======================
// TEXTURED GROUND
// =======================

glBindTexture(
    GL_TEXTURE_2D,
    grassTexture
);

glBegin(GL_QUADS);

glColor3f(1.0f, 1.0f, 1.0f);

glTexCoord2f(0.0f, 0.0f);
glVertex3f(-50.0f, -2.0f, -50.0f);

glTexCoord2f(20.0f, 0.0f);
glVertex3f( 50.0f, -2.0f, -50.0f);

glTexCoord2f(20.0f, 20.0f);
glVertex3f( 50.0f, -2.0f,  50.0f);

glTexCoord2f(0.0f, 20.0f);
glVertex3f(-50.0f, -2.0f,  50.0f);

glEnd();

// =======================
// TREES
// =======================

for (int i = -10; i <= 10; i += 5)
{
    // TREE TRUNK
    glPushMatrix();

    glTranslatef(
        i,
        -1.0f,
        -8.0f
    );

    glScalef(
        0.5f,
        2.0f,
        0.5f
    );

    glBegin(GL_QUADS);

    glColor3f(
        0.55f,
        0.27f,
        0.07f
    );

    // Front
    glVertex3f(-0.5f, -0.5f,  0.5f);
    glVertex3f( 0.5f, -0.5f,  0.5f);
    glVertex3f( 0.5f,  0.5f,  0.5f);
    glVertex3f(-0.5f,  0.5f,  0.5f);

    // Back
    glVertex3f(-0.5f, -0.5f, -0.5f);
    glVertex3f(-0.5f,  0.5f, -0.5f);
    glVertex3f( 0.5f,  0.5f, -0.5f);
    glVertex3f( 0.5f, -0.5f, -0.5f);

    glEnd();

    glPopMatrix();

    // LEAVES
    glPushMatrix();

    glTranslatef(
        i,
        1.5f,
        -8.0f
    );

    glScalef(
        2.0f,
        2.0f,
        2.0f
    );

    glBegin(GL_QUADS);

    glColor3f(
        0.1f,
        0.8f,
        0.1f
    );

    // Front
    glVertex3f(-0.5f, -0.5f,  0.5f);
    glVertex3f( 0.5f, -0.5f,  0.5f);
    glVertex3f( 0.5f,  0.5f,  0.5f);
    glVertex3f(-0.5f,  0.5f,  0.5f);

    // Back
    glVertex3f(-0.5f, -0.5f, -0.5f);
    glVertex3f(-0.5f,  0.5f, -0.5f);
    glVertex3f( 0.5f,  0.5f, -0.5f);
    glVertex3f( 0.5f, -0.5f, -0.5f);

    glEnd();

    glPopMatrix();
}
// =======================
// STARS
// =======================

glPointSize(3.0f);

glBegin(GL_POINTS);

glColor3f(
    1.0f,
    1.0f,
    1.0f
);

for (int i = -40; i <= 40; i += 4)
{
    for (int j = 5; j <= 25; j += 4)
    {
        glVertex3f(
            i,
            j,
            -50.0f
        );
    }
}

glEnd();

// =======================
// SUN
// =======================

glPushMatrix();

glTranslatef(
    15.0f,
    12.0f,
    -30.0f
);

glScalef(
    3.0f,
    3.0f,
    3.0f
);

glBegin(GL_POLYGON);

glColor3f(
    0.8f,
    0.8f,
    0.9f
);

for (int i = 0; i < 100; i++)
{
    float angle =
        2.0f * 3.14159f * i / 100;

    float x =
        cos(angle);

    float y =
        sin(angle);

    glVertex3f(
        x,
        y,
        0.0f
    );
}

glEnd();

glPopMatrix();

// =======================
// CLOUDS
// =======================

for (int i = -20; i <= 20; i += 10)
{
    glPushMatrix();

    glTranslatef(
        i + cloudMove,
        10.0f,
        -20.0f
    );

    glScalef(
        3.0f,
        1.5f,
        1.5f
    );

    glBegin(GL_QUADS);

    glColor3f(
        1.0f,
        1.0f,
        1.0f
    );

    // Front
    glVertex3f(-1, -1,  1);
    glVertex3f( 1, -1,  1);
    glVertex3f( 1,  1,  1);
    glVertex3f(-1,  1,  1);

    // Back
    glVertex3f(-1, -1, -1);
    glVertex3f(-1,  1, -1);
    glVertex3f( 1,  1, -1);
    glVertex3f( 1, -1, -1);

    glEnd();

    glPopMatrix();
}
// =======================
// MOUNTAINS
// =======================

for (int i = -30; i <= 30; i += 10)
{
    glPushMatrix();

    glTranslatef(
        i,
        3.0f,
        -25.0f
    );

    glScalef(
        5.0f,
        8.0f,
        5.0f
    );

    glBegin(GL_TRIANGLES);

    glColor3f(
        0.5f,
        0.5f,
        0.5f
    );

    // Front
    glVertex3f( 0.0f,  1.0f,  0.0f);
    glVertex3f(-1.0f, -1.0f,  1.0f);
    glVertex3f( 1.0f, -1.0f,  1.0f);

    // Right
    glVertex3f( 0.0f,  1.0f,  0.0f);
    glVertex3f( 1.0f, -1.0f,  1.0f);
    glVertex3f( 1.0f, -1.0f, -1.0f);

    // Back
    glVertex3f( 0.0f,  1.0f,  0.0f);
    glVertex3f( 1.0f, -1.0f, -1.0f);
    glVertex3f(-1.0f, -1.0f, -1.0f);

    // Left
    glVertex3f( 0.0f,  1.0f,  0.0f);
    glVertex3f(-1.0f, -1.0f, -1.0f);
    glVertex3f(-1.0f, -1.0f,  1.0f);

    glEnd();

    glPopMatrix();
}


        // =======================
        // ROTATING CUBE
        // =======================

        glPushMatrix();

        glRotatef(
            rotation,
            1.0f,
            1.0f,
            0.0f
        );

        glBegin(GL_QUADS);

        // FRONT
        glColor3f(1, 0, 0);

        glVertex3f(-1, -1,  1);
        glVertex3f( 1, -1,  1);
        glVertex3f( 1,  1,  1);
        glVertex3f(-1,  1,  1);

        // BACK
        glColor3f(0, 1, 0);

        glVertex3f(-1, -1, -1);
        glVertex3f(-1,  1, -1);
        glVertex3f( 1,  1, -1);
        glVertex3f( 1, -1, -1);

        // TOP
        glColor3f(0, 0, 1);

        glVertex3f(-1,  1, -1);
        glVertex3f(-1,  1,  1);
        glVertex3f( 1,  1,  1);
        glVertex3f( 1,  1, -1);

        // BOTTOM
        glColor3f(1, 1, 0);

        glVertex3f(-1, -1, -1);
        glVertex3f( 1, -1, -1);
        glVertex3f( 1, -1,  1);
        glVertex3f(-1, -1,  1);

        // RIGHT
        glColor3f(1, 0, 1);

        glVertex3f(1, -1, -1);
        glVertex3f(1,  1, -1);
        glVertex3f(1,  1,  1);
        glVertex3f(1, -1,  1);

        // LEFT
        glColor3f(0, 1, 1);

        glVertex3f(-1, -1, -1);
        glVertex3f(-1, -1,  1);
        glVertex3f(-1,  1,  1);
        glVertex3f(-1,  1, -1);

        glEnd();

        glPopMatrix();

        // =======================
        // IMGUI RENDER
        // =======================

        ImGui_ImplOpenGL3_RenderDrawData(
            ImGui::GetDrawData()
        );

        glfwSwapBuffers(window);
    }

    // CLEANUP
    ImGui_ImplOpenGL3_Shutdown();

    ImGui_ImplGlfw_Shutdown();

    ImGui::DestroyContext();

    glfwDestroyWindow(window);

    glfwTerminate();

    return 0;
}
