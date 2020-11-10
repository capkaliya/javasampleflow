# javasampleflow

@WebServlet(name = "ServletFileUpload", urlPatterns = {"/ServletFileUpload"})
@MultipartConfig(fileSizeThreshold = 1024 * 1024 * 10, // 10 MB
        maxFileSize = 1024 * 1024 * 1000, // 1 GB
        maxRequestSize = 1024 * 1024 * 1000)   	// 1 GB
public class ServletFileUpload extends HttpServlet {

    PrintWriter out;
    Connection con;
    PreparedStatement ps;
    ServletOutputStream os;

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/plain;charset=UTF-8");
        try {
            //out = response.getWriter();//.........do not remove comment.
            os = response.getOutputStream();
            String folderName = "resources";
            String uploadPath = request.getServletContext().getRealPath("") + File.separator + folderName;//for netbeans use this code
            //String uploadPath = request.getServletContext().getRealPath("") + folderName;//for eclipse use this code
            File dir = new File(uploadPath);
            if (!dir.exists()) {
                dir.mkdirs();
            }
            Part filePart = request.getPart("file");
            int id = Integer.parseInt(request.getParameter("id"));
            String firstName = request.getParameter("firstname");
            String lastName = request.getParameter("lastname");
            String fileName = filePart.getSubmittedFileName();
            String path = folderName + File.separator + fileName;
            Timestamp added_date = new Timestamp(System.currentTimeMillis());
            System.out.println("fileName: " + fileName);
            System.out.println("Path: " + uploadPath);
            System.out.println("Name: " + firstName);
            InputStream is = filePart.getInputStream();
            Files.copy(is, Paths.get(uploadPath + File.separator + fileName), StandardCopyOption.REPLACE_EXISTING);

            try {
                con = DB.getConnection();
                System.out.println("connection done");
                String sql = "insert into employee values(?,?,?,?,?,?)";
                ps = con.prepareStatement(sql);
                ps.setInt(1, id);
                ps.setString(2, firstName);
                ps.setString(3, lastName);
                ps.setString(4, fileName);
                ps.setString(5, path);
                ps.setTimestamp(6, added_date);
                int status = ps.executeUpdate();
                if (status > 0) {
                    os.println("File uploaded successfully...");
                    os.println("Uploaded Path: " + uploadPath);
                }
            } catch (SQLException e) {
                os.println("Some error occured please console log.");
                System.out.println("Exception1: " + e);
            }

        } catch (IOException | ServletException e) {
            os.println("Some error occured please console log.");
            System.out.println("Exception2: " + e);
        }
    }

}
