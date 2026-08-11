${AnsiColor.BRIGHT_RED}
/ \        ${AnsiColor.BRIGHT_CYAN} _                      _____             _
/   \       ${AnsiColor.BRIGHT_CYAN}| |                    |  ___|           |_|
/  O  \      ${AnsiColor.BRIGHT_CYAN}| |      __ _  ____ ___| |__  _ __   __ _ _ _ __   ___  ___ _ __
/  [_]  \     ${AnsiColor.BRIGHT_CYAN}| |     / _` ||_  //_  /  __| | '_ \ / _` | | '_ \ / _ \/ _ \ '__|
/  (o.o)  \    ${AnsiColor.BRIGHT_CYAN}| |____| (_| | / /  / /| |____| | | | (_| | | | | |  __/  __/ |
(___/   \___)   ${AnsiColor.BRIGHT_CYAN}|______|\__,_|/___|/___|______|_| |_|\__, |_|_| |_|\___|\___|_|
/| |===| |\                                          __/ |
(_| |___| |_)                                        |___/
${AnsiColor.BRIGHT_CYAN}

╔══════════════════════════════════════════════════════════════════════════════════════════╗
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_WHITE}${AnsiStyle.BOLD}   🚀  APPLICATION STARTUP                                                                ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_GREEN}${AnsiStyle.BOLD}   📋  APPLICATION                                                                        ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Application   ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${spring.application.name:-Spring Boot Application}                                                 ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Version       ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${application.version:-1.0.0-SNAPSHOT}                                                      ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Spring Boot   ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${spring-boot.version}                                                                ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Java          ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${java.version}                                                               ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Profile       ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_YELLOW}${spring.profiles.active:-default}                                                                  ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Process ID    ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${PID}                                                               ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Started By    ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${user.name}                                                       ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_CYAN}${AnsiStyle.BOLD}   🌐  SERVER                                                                             ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Server URL    ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;http://localhost:${server.port:-8080}${server.servlet.context-path:-}\http://localhost:${server.port:-8080}${server.servlet.context-path:-}]8;;\                                               ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Protocol      ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}HTTP/1.1                                                             ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Context Path  ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${server.servlet.context-path:-/}                                                                   ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Swagger UI    ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;http://localhost:${server.port:-8080}/swagger-ui.html\http://localhost:${server.port:-8080}/swagger-ui.html]8;;\                                ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ API Docs      ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;http://localhost:${server.port:-8080}/v3/api-docs\http://localhost:${server.port:-8080}/v3/api-docs]8;;\                                    ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Health Check  ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;http://localhost:${server.port:-8080}/actuator/health\http://localhost:${server.port:-8080}/actuator/health]8;;\                                ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_CYAN}${AnsiStyle.BOLD}   💻  OPERATING SYSTEM & ENVIRONMENT                                                     ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ OS Name       ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${os.name}                                                                ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ OS Version    ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${os.version}                                                     ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Architecture  ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${os.arch}                                                                ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ User Home     ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${user.home}                                                 ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_MAGENTA}${AnsiStyle.BOLD}   🔌  INFRASTRUCTURE                                                                     ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.BRIGHT_YELLOW}● ${AnsiColor.MAGENTA}Database      ${AnsiColor.BRIGHT_BLUE}::   ${AnsiColor.BRIGHT_YELLOW}CHECKING...                                                        ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.BRIGHT_YELLOW}● ${AnsiColor.MAGENTA}Redis         ${AnsiColor.BRIGHT_BLUE}::   ${AnsiColor.BRIGHT_YELLOW}CHECKING...                                                        ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.BRIGHT_YELLOW}● ${AnsiColor.MAGENTA}Kafka         ${AnsiColor.BRIGHT_BLUE}::   ${AnsiColor.BRIGHT_YELLOW}CHECKING...                                                        ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_YELLOW}${AnsiStyle.BOLD}   ⚙  RUNTIME                                                                             ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.YELLOW}◆ JVM           ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${java.vm.name}                                             ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.YELLOW}◆ JVM Version   ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${java.vm.version}                                            ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.YELLOW}◆ Java Home     ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${java.home}                                   ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.YELLOW}◆ OS            ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${os.name} ${os.version}                                               ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.YELLOW}◆ Architecture  ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${os.arch}                                                                ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.YELLOW}◆ Timezone      ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${user.timezone}                                                           ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.YELLOW}◆ File Encoding ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${file.encoding}                                                                ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_RED}${AnsiStyle.BOLD}   🔒  SECURITY & ACCESS CONTROL                                                          ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ Auth Type     ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}JWT / OAuth2 Bearer Token                                            ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ Security Doc  ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;http://localhost:${server.port:-8080}/actuator/env\Spring Security Active]8;;\                                               ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ CORS Status   ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_GREEN}ENABLED                                                              ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_CYAN}${AnsiStyle.BOLD}   📦  BUILD INFO                                                                         ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Build Version ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${application.version:-Development}                                                         ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Environment   ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${spring.profiles.active:-default}                                                                  ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Commit        ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${git.commit.id.abbrev:-N/A}                                                                 ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Branch        ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${git.branch:-N/A}                                                                 ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.CYAN}◆ Build Time    ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}${git.build.time:-N/A}                                                                 ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_RED}${AnsiStyle.BOLD}   👨‍💻  DEVELOPER                                                                       ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ Developer     ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}Parvez Hossain                                                       ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ Alias         ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}@Lazyengineer                                                        ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ Email         ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;mailto:parvezhossain724@gmail.com\parvezhossain724@gmail.com]8;;\                                           ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ GitHub        ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;https://github.com/ParvezHossain\github.com/ParvezHossain]8;;\                                             ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ LinkedIn      ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;https://linkedin.com/in/parvez-hossain\linkedin.com/in/parvez-hossain]8;;\                                       ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.RED}◆ X / Twitter   ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;https://x.com/parvez__hossain\x.com/parvez__hossain]8;;\                                                ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_GREEN}${AnsiStyle.BOLD}   🔗  QUICK LINKS                                                                        ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Local App     ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;http://localhost:${server.port:-8080}${server.servlet.context-path:-}\http://localhost:${server.port:-8080}${server.servlet.context-path:-}]8;;\                                               ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Swagger UI    ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;http://localhost:${server.port:-8080}/swagger-ui.html\Open API Explorer]8;;\                                                    ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Actuator      ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;http://localhost:${server.port:-8080}/actuator\Health / Metrics]8;;\                                                     ${AnsiColor.BRIGHT_BLUE}║
║  ${AnsiColor.GREEN}◆ Repository    ${AnsiColor.BRIGHT_BLUE}:: ${AnsiColor.BRIGHT_WHITE}]8;;https://github.com/ParvezHossain\View Source on GitHub]8;;\                                                ${AnsiColor.BRIGHT_BLUE}║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║${AnsiColor.BRIGHT_BLUE}${AnsiColor.BRIGHT_GREEN}${AnsiStyle.BOLD}   ✓  APPLICATION INITIALIZED                                                             ${AnsiStyle.NORMAL}${AnsiBackground.DEFAULT}${AnsiColor.BRIGHT_BLUE}║
╚══════════════════════════════════════════════════════════════════════════════════════════╝

${AnsiColor.BRIGHT_CYAN}
:: Welcome to ${spring.application.name:-Spring Boot} ::
${AnsiColor.DEFAULT}